# Setup

SWC is a part of the new generation of js/ts compilers.

In this example it is used with cdktf to improve build times.

- [Setup](#setup)
  - [Conclusion](#conclusion)
    - [Synth](#synth)
    - [Plan](#plan)
  - [Create Project](#create-project)
  - [Setup Resources](#setup-resources)
  - [Start compiling](#start-compiling)
    - [Optimization OPtions](#optimization-options)
      - [Enable incremental ts builds](#enable-incremental-ts-builds)
      - [Use SWC to transpile .ts to commonjs module](#use-swc-to-transpile-ts-to-commonjs-module)


## Conclusion

- Moving from tsc makes transpiling 6x faster
- Type-definitions are lost
  - Only an issue if you plan on packaging to distribute; not a problem for this use case.

### Synth

'COMMAND=compile yarn synth' ran
  5.62 ± 0.82 times faster than 'COMMAND=build yarn synth'

| Command                      |       Mean [s] | Min [s] | Max [s] |    Relative |
| :--------------------------- | -------------: | ------: | ------: | ----------: |
| `COMMAND=compile yarn synth` |  4.984 ± 0.251 |   4.768 |   5.259 |        1.00 |
| `COMMAND=build yarn synth`   | 28.016 ± 3.812 |  24.929 |  32.277 | 5.62 ± 0.82 |


### Plan


## Create Project

```bash
mkdir cdktf-swc

npx cdktf init --template="typescript" --local
```

## Setup Resources

Follow this tutorial to configure an EC2 instance.

## Start compiling

With this all set up we can run:

```bash
yarn synth
```

This is a little slow:

```bash
time cdktf synth
Generated Terraform code for the stacks: typescript-aws
Done in 32.13s.

real    0m33.510s
user    0m53.192s
sys     0m1.894s
```

### Optimization OPtions

#### Enable incremental ts builds

Stores a build manifest after first build:

```bash
$ time yarn synth
yarn run v1.22.5
$ cdktf synth
Generated Terraform code for the stacks: typescript-aws
Done in 15.45s.

real    0m15.728s
user    0m28.983s
sys     0m1.013s
```

#### Use SWC to transpile .ts to commonjs module

```bash
yarn add --dev @swc/core @swc/cli
```

```js
// .swcrc
{
  "jsc": {
    "parser": {
      "syntax": "typescript"
    },
    "target": "es2018"
  },
  "sourceMaps": true,
  "module": {
    "type": "commonjs",
    "strict": false,
    "noInterop": false,
    "strictMode": false
  }
}
```

Update the compile script in package.json:

```jsonc
{
  "scripts": [
    "compile": "swc main.ts --out-file dist/main.js & swc .gen/providers/**/*.ts -d dist/.gen/"
  ]
}
```

```bash
$ time yarn synth
yarn run v1.22.5
$ cdktf synth
Successfully compiled 1 file with swc.

Successfully compiled: 2088 files with swc (681.19ms)

Generated Terraform code for the stacks: typescript-aws
Done in 5.12s.

real    0m5.389s
user    0m16.437s
sys     0m1.046s
```
