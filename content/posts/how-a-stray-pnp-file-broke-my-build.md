+++
title = 'How a Stray .pnp.cjs File Broke My Build'
date = 2025-02-27T11:34:04+01:00
draft = false
tags = [
  "PnP",
  "pnpm",
  "YarnPnP",
  "JavaScript",
  "TypeScript",
  "NodeJS",
  "DependencyManagement",
]
+++

Ever had a build fail with cryptic errors about missing dependencies, even though you *know* the packages are installed? If you're using `pnpm`, the issue might be a stray `.pnp.cjs` file lurking in an unexpected place <del>because your past self did something dumb<del>.

### What Happened?
While working on a recently cloned project with `pnpm`, my builds where failing with errors about unresolved packages like `chalk`, `hanji`, and `zod`. The logs pointed to Yarn PnP, which was odd because I wasn’t using Yarn. After some digging, I found a `.pnp.cjs` file in my `~/git/` folder. This file was interfering with `pnpm` somehow, making it act like it was in a Yarn PnP environment.

This is a slice of the error in question:

```bash
...
drizzle-kit:build:   You can mark the path "zod" as external to exclude it from the bundle, which will remove this
drizzle-kit:build:   error and leave the unresolved path in the bundle.
drizzle-kit:build: 
drizzle-kit:build: ✘ [ERROR] Could not resolve "zod"
drizzle-kit:build: 
drizzle-kit:build:     src/serializer/gelSchema.ts:3:110:
drizzle-kit:build:       3 │ import { any, array, boolean, enum as enumType, literal, number, object, record, string, TypeOf, union } from 'zod';
drizzle-kit:build:         ╵                                                                                                               ~~~~~
drizzle-kit:build: 
drizzle-kit:build:   The Yarn Plug'n'Play manifest forbids importing "zod" here because it's not listed as a dependency
drizzle-kit:build:   of this package:
drizzle-kit:build: 
drizzle-kit:build:     ../../.pnp.cjs:37:31:
drizzle-kit:build:       37 │         "packageDependencies": [\
drizzle-kit:build:          ╵                                ~~
drizzle-kit:build: 
drizzle-kit:build:   You can mark the path "zod" as external to exclude it from the bundle, which will remove this
drizzle-kit:build:   error and leave the unresolved path in the bundle.
drizzle-kit:build: 
drizzle-kit:build: node:internal/process/esm_loader:108
drizzle-kit:build:     internalBinding('errors').triggerUncaughtException(
drizzle-kit:build:                               ^
drizzle-kit:build: 
drizzle-kit:build: Error: Build failed with 8 errors:
drizzle-kit:build: src/cli/views.ts:1:18: ERROR: Could not resolve "chalk"
drizzle-kit:build: src/cli/views.ts:2:54: ERROR: Could not resolve "hanji"
drizzle-kit:build: src/serializer/gelSchema.ts:3:110: ERROR: Could not resolve "zod"
drizzle-kit:build: src/serializer/mysqlSchema.ts:1:95: ERROR: Could not resolve "zod"
drizzle-kit:build: src/serializer/pgSchema.ts:3:110: ERROR: Could not resolve "zod"
drizzle-kit:build: ...
drizzle-kit:build:     at failureErrorWithLog (/Users/cipriano/git/drizzle-orm/node_modules/.pnpm/esbuild@0.19.12/node_modules/esbuild/lib/main.js:1651:15)
drizzle-kit:build:     at /Users/cipriano/git/drizzle-orm/node_modules/.pnpm/esbuild@0.19.12/node_modules/esbuild/lib/main.js:1059:25
drizzle-kit:build:     at /Users/cipriano/git/drizzle-orm/node_modules/.pnpm/esbuild@0.19.12/node_modules/esbuild/lib/main.js:1004:52
drizzle-kit:build:     at buildResponseToResult (/Users/cipriano/git/drizzle-orm/node_modules/.pnpm/esbuild@0.19.12/node_modules/esbuild/lib/main.js:1057:7)
drizzle-kit:build:     at /Users/cipriano/git/drizzle-orm/node_modules/.pnpm/esbuild@0.19.12/node_modules/esbuild/lib/main.js:1086:16
drizzle-kit:build:     at responseCallbacks.<computed> (/Users/cipriano/git/drizzle-orm/node_modules/.pnpm/esbuild@0.19.12/node_modules/esbuild/lib/main.js:704:9)
drizzle-kit:build:     at handleIncomingPacket (/Users/cipriano/git/drizzle-orm/node_modules/.pnpm/esbuild@0.19.12/node_modules/esbuild/lib/main.js:764:9)
drizzle-kit:build:     at Socket.readFromStdout (/Users/cipriano/git/drizzle-orm/node_modules/.pnpm/esbuild@0.19.12/node_modules/esbuild/lib/main.js:680:7)
drizzle-kit:build:     at Socket.emit (node:events:517:28)
drizzle-kit:build:     at addChunk (node:internal/streams/readable:335:12) {
drizzle-kit:build:   errors: [
drizzle-kit:build:     {
drizzle-kit:build:       detail: undefined,
drizzle-kit:build:       id: '',
drizzle-kit:build:       location: {
drizzle-kit:build:         column: 18,
drizzle-kit:build:         file: 'src/cli/views.ts',
drizzle-kit:build:         length: 7,
drizzle-kit:build:         line: 1,
drizzle-kit:build:         lineText: "import chalk from 'chalk';",
drizzle-kit:build:         namespace: '',
drizzle-kit:build:         suggestion: ''
drizzle-kit:build:       },
drizzle-kit:build:       notes: [
drizzle-kit:build:         {
drizzle-kit:build:           location: {
drizzle-kit:build:             column: 31,
drizzle-kit:build:             file: '../../.pnp.cjs',
drizzle-kit:build:             length: 64,
drizzle-kit:build:             line: 37,
drizzle-kit:build:             lineText: '        "packageDependencies": [\\',
drizzle-kit:build:             namespace: '',
drizzle-kit:build:             suggestion: ''
drizzle-kit:build:           },
drizzle-kit:build:           text: `The Yarn Plug'n'Play manifest forbids importing "chalk" here because it's not listed as a dependency of this package:`
drizzle-kit:build:         },
drizzle-kit:build:         {
drizzle-kit:build:           location: null,
drizzle-kit:build:           text: 'You can mark the path "chalk" as external to exclude it from the bundle, which will remove this error and leave the unresolved path in the bundle.'
drizzle-kit:build:         }
drizzle-kit:build:       ],
drizzle-kit:build:       pluginName: '',
drizzle-kit:build:       text: 'Could not resolve "chalk"'
drizzle-kit:build:     },
drizzle-kit:build:     {
drizzle-kit:build:       detail: undefined,
drizzle-kit:build:       id: '',
drizzle-kit:build:       location: {
drizzle-kit:build:         column: 54,
drizzle-kit:build:         file: 'src/cli/views.ts',
drizzle-kit:build:         length: 7,
drizzle-kit:build:         line: 2,
drizzle-kit:build:         lineText: "import { Prompt, render, SelectState, TaskView } from 'hanji';",
drizzle-kit:build:         namespace: '',
drizzle-kit:build:         suggestion: ''
drizzle-kit:build:       },
drizzle-kit:build:       notes: [
drizzle-kit:build:         {
drizzle-kit:build:           location: {
drizzle-kit:build:             column: 31,
drizzle-kit:build:             file: '../../.pnp.cjs',
drizzle-kit:build:             length: 64,
drizzle-kit:build:             line: 37,
drizzle-kit:build:             lineText: '        "packageDependencies": [\\',
drizzle-kit:build:             namespace: '',
drizzle-kit:build:             suggestion: ''
drizzle-kit:build:           },
drizzle-kit:build:           text: `The Yarn Plug'n'Play manifest forbids importing "hanji" here because it's not listed as a dependency of this package:`
drizzle-kit:build:         },
drizzle-kit:build:         {
drizzle-kit:build:           location: null,
drizzle-kit:build:           text: 'You can mark the path "hanji" as external to exclude it from the bundle, which will remove this error and leave the unresolved path in the bundle.'
drizzle-kit:build:         }
drizzle-kit:build:       ],
drizzle-kit:build:       pluginName: '',
drizzle-kit:build:       text: 'Could not resolve "hanji"'
drizzle-kit:build:     },
...
```

### How I solved it
By the error logs, I could see that the `.pnp.cjs` file was causing the issue, but I had no .pnp.cjs file in my project directory. 

Then, I used the `find` command to scan my home directory for any `.pnp.*` files:

```bash
find ~ -name ".pnp.*"
```

If you're on Mac, it will start to ask for permission to access some directories, just grant it. The command will start to print some directories where the operation is not permitted, but at some point (hopefully) it will print the directories where it found the `.pnp.*` files.

```bash
find: /Users/cipriano/Library/Accounts: Operation not permitted
find: /Users/cipriano/Library/Safari: Operation not permitted
find: /Users/cipriano/Library/Biome: Operation not permitted
find: /Users/cipriano/.Trash: Operation not permitted
/Users/cipriano/git/.pnp.cjs
/Users/cipriano/git/unrelated-project/.pnp.cjs
```

If you have some yarn projects in your home directory, you might see some `.pnp.*` files in those directories, you can ignore them. But this also revealed the stray `.pnp.cjs` file hiding in `~/git/`.

### But why?
Yarn PnP uses `.pnp.cjs` to enforce strict dependency resolution. If this file exists in a parent directory (like `~/git/` or your home directory), it can accidentally affect other tools like `pnpm`, causing them to apply Yarn PnP's rules.

### The Fix
I deleted the `.pnp.cjs` file, and the build worked perfectly:

```bash
rm ~/git/.pnp.cjs
```

### So...
If your build suddenly breaks, check for stray `.pnp.cjs` files in parent directories. Sometimes, the small things can make you disappointed.
