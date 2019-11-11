# rush-pnpm-4-bug

Reproduction of a bug with Rush + pnpm: https://github.com/microsoft/rushstack/issues/1623

When used with pnpm@4, Rush does not respect the pinned version of a transitive dependency. Instead, the latest compatible version is installed.

### The issue

This repository uses Rush with one project: [`mypackage`](packages/mypackage). `mypackage` depends on npm package `@elliottsj/third-party-direct-dependency@2.0.1`, which itself depends on `@elliottsj/third-party-transitive-dependency@^3.0.0`:

```
mypackage
  -> @elliottsj/third-party-direct-dependency@2.0.1
    -> @elliottsj/third-party-transitive-dependency@^3.0.0
```

Note the semver constraint of `^3.0.0` rather than a specific version.

There is one published version of `@elliottsj/third-party-direct-dependency` and two of `@elliottsj/third-party-transitive-dependency`:

```sh-session
$ npm view @elliottsj/third-party-direct-dependency versions
[ '2.0.1' ]
```

```sh-session
$ npm view @elliottsj/third-party-transitive-dependency versions
[ '3.0.0', '3.1.0' ]
```

The lockfile [`pnpm-lock.yaml`](common/config/rush/pnpm-lock.yaml) was generated _before `@elliottsj/third-party-transitive-dependency@3.1.0` was published_, so it specifies version 3.0.0 exactly:

```yaml
packages:
  /@elliottsj/third-party-direct-dependency/2.0.1:
    dependencies:
      '@elliottsj/third-party-transitive-dependency': 3.0.0
```

Now that version 3.1.0 is published, the problem is that `rush install` installs version 3.1.0 of `@elliottsj/third-party-transitive-dependency` rather than 3.0.0 as specified in the lockfile:

```sh-session
$ rush install -p
...
$ cd common/temp/node_modules/@elliottsj/third-party-direct-dependency/
$ node -p "require(\"@elliottsj/third-party-transitive-dependency/package.json\").version"
3.1.0
```

### Using pnpm without Rush

The problem doesn't seem to occur when using pnpm standalone:

```sh-session
$ cd pnpm-only/
$ pnpm install
...
$ cd node_modules/@elliottsj/third-party-direct-dependency/
$ node -p "require(\"@elliottsj/third-party-transitive-dependency/package.json\").version"
3.0.0
```

### Using pnpm@3

The problem doesn't occur when using pnpm@3.8.1. Check out the [pnpm-3 branch](https://github.com/elliottsj/rush-pnpm-4-bug/tree/pnpm-3):

```sh-session
$ rush install -p
...
$ cd common/temp/node_modules/@elliottsj/third-party-direct-dependency/
$ node -p "require(\"@elliottsj/third-party-transitive-dependency/package.json\").version"
3.0.0
```
