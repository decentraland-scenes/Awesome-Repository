# Library migration guide

- Last updated Feb 10 2021

The utils libraries have undergone some changes to how they must be imported. These changes make them significantly lighter in the exported scenes, and make their import paths a lot friendlier to reach, keeping a same address from anywhere in the project.

The affected libraries are:

- [ECS Utils](https://www.npmjs.com/package/decentraland-ecs-utils)
- [UI Utils](https://github.com/decentraland/decentraland-ui-utils)
- [NPC Utils](https://github.com/decentraland/decentraland-npc-utils)
- [Crypto Utils](https://github.com/decentraland/decentraland-crypto-utils)
- [Layer 2 Utils](https://github.com/decentraland/decentraland-l2-utils)
- [Noise Utils](https://github.com/decentraland/decentraland-noise-utils)

To update your projects to use the latest versions of these libraries:

1. Make sure you have the latest SDK version installed

  ```
  npm i decentraland-ecs@latest
  ```

  > Note: This is an essential step if your project uses two or more of these libraries. This latest release includes a fix that prevents the namespaces between libraries getting mixed.

2. If installing into a project where you already had older versions of these libraries, remove the old versions of the libraries from your project.

  ```
  // ECS utils
  npm un decentraland-ecs-utils

  // UI utils
  npm un @dcl/ui-utils

  // NPC utils
  npm un @dcl/npc-utils

  // Crypto utils
  npm un @dcl/crypto-utils

  // Layer 2 utils
  npm un @dcl/l2-utils

  // Noise utils
  npm un @dcl/noise-utils
  ```

  > Note: Since the new libraries are named differently, you will otherwise keep both versions in your project, which may lead to errors.

3. Install the new versions of the libraries that apply to your project:

  ```
  // ECS utils
  npm i @dcl/ecs-scene-utils -B

  // UI utils
  npm i @dcl/ui-scene-utils -B

  // NPC utils
  npm i @dcl/npc-scene-utils -B

  // Crypto utils
  npm i @dcl/crypto-scene-utils  @dcl/ecs-scene-utils eth-connect -B

  // Layer 2 utils
  npm i '@dcl/l2-scene-utils @dcl/ecs-scene-utils eth-connect -B

  // Noise utils
  npm i @dcl/noise-utils -B
  ```

	> Note: The Crypto Utils and Layer 2 utils both require to also have installed the eth-connect library and the ECS Utils library.

4. Run `dcl start` or `dcl build`. This will build the required internal folders inside the dependencies.

  > Note: Before you do this step, it's normal for VS Studio Code (or any other IDE you use) to mark the imports as broken.

5. Change the lines that import the libraries into your .ts files to the following:

  ```
  // ECS utils
  import * as utils from ‘@dcl/ecs-scene-utils'

  // UI utils
  import * as ui from '@dcl/ui-scene-utils'

  // NPC utils
  import { NPC } from '@dcl/npc-scene-utils'

  // Crypto utils
  import * as crypto from '@dcl/crypto-scene-utils'

  // L2 utils
  import * as layerTwo from '@dcl/l2-scene-utils'

  // Noise utils
  import { Noise } from '@dcl/noise-utils'
  ```

## Troubleshooting

#### Avoid importing from /node_modules directly

Make sure you aren't doing additional imports of types or objects from these libraries via a path in the `node_modules` folder.

For example, with the `ui-utils` library, it was customary to do additional imports like these:

```ts
import {
  BarStyles,
  PromptStyles,
} from '../node_modules/@dcl/ui-utils/utils/types'
```

Or on the `ecs-utils` library, it was customary to do additional imports like these:

```ts
import {
  TriggerBoxShape,
  TriggerSphereShape,
} from '../node_modules/decentraland-ecs-utils/triggers/triggerSystem'
import { InterpolationType } from '../node_modules/decentraland-ecs-utils/transform/math/interpolation'
```

Or in the crptoutils library it was customary to import contract addresses like this:

```ts
import { mainnet } from '../node_modules/@dcl/crypto-utils/utils/contract'
```

Make sure you don't have any of those lying around in your project.

You can now import these elements together with everything else. 

For example if you imported the `ecs-utils` lib via `import * as utils from ‘@dcl/ecs-scene-utils'`, you can access _TriggerBoxShape_ as `utils.TriggerBoxShape` and _InterpolationType_ as `utils.InterpolationType`.

If you imported the `ui-utils` lib via ` import * as ui from '@dcl/ui-scene-utils'`, you can access _BarStyles_ as `ui.BarStyles` and _PromptStyles_ as `ui.PromptStyles`

If you imported the `crpto-utils` lib via `import * as crypto from '@dcl/crypto-scene-utils'`, you can access contract addresses as `crypto.contract.mainnet.<contract>`.

#### TriggerComponent syntax change

If you used the TriggerComponent in the ‘ecs-utils’ lib, you may also need to change the syntax. An earlier release changed that so that you only deal with the parameters that you need when creating a trigger.

So what used to look like this:

```ts
myEntity.addComponent(
    new utils.TriggerComponent(
        new utils.TriggerBoxShape(
          new Vector3(2,2,2),
          Vector3.Zero()
        ),
        0,
        0,
        null,
        null,
        () => {
           log('TRIGGERED!')
        },
        () => {
           log('LEFT!')
        },
    })
  )
```

Now looks like this:

```ts
myEntity.addComponent(
    new utils.TriggerComponent(
      new utils.TriggerBoxShape(),
      {
        onCameraEnter: () => {
          log('TRIGGERED!')
        }
        onCameraExit: () => {
          log('LEFT!')
        }
      }
    )
)
```

#### Dialog object only on NPC library

The `Dialog` object was available both in the `ui-utils` library and the `npc-utils` library with the exact same list of features. To avoid this redundancy, it's now only available as part of the `npc-utils` library, with all the same syntax.

If your scene used to rely on the UI Utils library for a Dialog window, install the NPC Utils library instead to handle this.


#### Crypto Utils and L2 need dependencies

If your scene is using the Crypto Utils or the Layer 2 Utils libraries, you also need to have eth-connect installed, and ECS Utils.

Prior versions of these libraries didn't need these installations to be done manually in the scene (but they installed them in their own `/node_modules` folders, adding significant redundant bulk to the scene code).

Make sure your project has the latest versions of both these dependencies:


```
npm i @dcl/ecs-scene-utils eth-connect -B
```

#### Overlapping paths

If in the past you attempted to install a bundled library (including eth-connect) into your scene using an SDK version that was older than _6.6.4_, you may encounter this problem. If the scene imports more than one bundled library, then the paths might get mixed up and misinterpreted.

To check for this issue, open the `tsconfig.json` file and look at the _paths_ field.

In a healthy scene, each library will have a distinct path, like this:

```json
"paths": {
      "@dcl/crypto-scene-utils": [
        "node_modules/@dcl/crypto-scene-utils/dist/index.d.ts"
      ],
      "@dcl/ecs-scene-utils": [
        "node_modules/@dcl/ecs-scene-utils/dist/index.d.ts"
      ],
      "eth-connect": [
        "node_modules/eth-connect/eth-connect.d.ts"
      ],
      "@dcl/ui-scene-utils": [
        "node_modules/@dcl/ui-scene-utils/dist/index.d.ts"
      ]
    }
```

In a corrupted scene, all libraries will be under the _*_ path, like this:

```json
  "paths": {
      "*": [
        "node_modules/@dcl/crypto-scene-utils/dist/index.d.ts",
        "node_modules/@dcl/ecs-scene-utils/dist/index.d.ts",
        "node_modules/eth-connect/eth-connect.d.ts",
        "node_modules/@dcl/ui-scene-utils/dist/index.d.ts"
      ]
    },
```

To fix this issue:

1) Make sure your scene has version 6.6.4 or later of the SDK

```
npm i decentraland-ecs@latest
```

2) Manually delete all of the _paths_ property from `tsconfig.json`. 

3) Run `dcl start` or `dcl build` for a new _paths_ property to get correctly written into the `tsconfig.json` file.
