# lib.nixosSystem

## pkgs
Do NOT set `pkgs` in `lib.nixosSystem`
there's this stupid pattern which comes from some old guide or youtube video which goes like this: 
```nix
let 
  system = "x86_64-linux";
  pkgs = import nixpkgs {
    inherit system;
    overlays = [];
  };
in
{
  nixosConfigurations."nixos" = lib.nixosSystem {
    inherit system pkgs;
    #           HERE^
    modules = [];
    specialArgs = {
        inherit pkgs;
    #     or HERE^
    };
  };
}
```
There is never a reason to do this and it will stop all `nixpkgs.` options from working SILENTLY WITHOUT ERROR
if you want to instantiate your own nixpkgs instance use the nixpkgs `readOnlyPkgs` [module](https://github.com/NixOS/nixpkgs/blob/ba10489eae3b2b2f665947b516e7043594a235c8/flake.nix#L61-L72)


## specialArgs
There is no `specialArgs` checking in `lib.nixosSystem`
you can pass `config`,`lib`,`pkgs`, or `modulesPath` just to name the main ones and completely break your config

Do not pass random values to `specialArgs` such as: 
- `hostname`: can be found under `config.networking.hostName` already
- `user`: just why?
- `stateVersion`: this should be hard-coded per-configuration so there's no use in passing it around

## system
You don't need to pass `system` to `lib.nixosSystem`
if you view the [source](https://github.com/NixOS/nixpkgs/blob/ba10489eae3b2b2f665947b516e7043594a235c8/nixos/lib/eval-config.nix#L12-L15) for `lib.nixosSystem`
you'll see it straight up says: 
> this can be set modularly, it would be nice to remove
and if you use a generated hardware-configuration you already have a line like this 
```nix
nixpkgs.hostPlatform = lib.mkDefault "x86_64-linux";
```
if you don't have that line or are making an abstraction over `lib.nixosSystem`
simply turn 
```nix
lib.nixosSystem {
  system = "x86_64-linux";
  modules = [
...
```
into 
```nix
lib.nixosSystem {
  modules = [
  {
    nixpkgs.hostPlatform = "x86_64-linux";
  }
```

## Ideal case 
IMO this is what an ideal setup looks like (unless you're abstracting host creation):
```nix
lib.nixosSystem {
  modules = [
    # A single module entry point
    ./hosts/nixos
  ];
  # Pass inputs to your configuration for easy use
  specialArgs = { inherit inputs;};
}
```
## it's stupid
It's just a shallow wrapper around `lib.evalModules` and you can easily use `lib.evalModules` directly for example: 
```nix
lib.evalModules {
  modules = [
    ./hosts/nixos
    (import "${nixpkgs}/nixos/modules/module-list.nix")
  ];
  specialArgs = {
    inherit inputs;
    modulesPath = "${nixpkgs}/nixos/modules";
  };
};

```
should be identical to the "Ideal case"

