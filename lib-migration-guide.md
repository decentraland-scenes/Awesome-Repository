# Library migration guide

- Last updated Feb 10 2021

The utils libraries have undergone some changes to how they must be imported. These changes make them significantly lighter in the exported scenes, and make their import paths a lot friendlier to reach, keeping a same address from anywhere in the project.

The affected libraries are:

- [ECS Utils](https://www.npmjs.com/package/decentraland-ecs-utils)
- [UI Utils](https://github.com/decentraland/decentraland-ui-utils)
- [NPC Utils](https://github.com/decentraland/decentraland-npc-utils)
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

  // Noise utils
  npm i @dcl/noise-scene-utils -B
  ```

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

  // Noise utils
  import { Noise } from '@dcl/noise-utils
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

Make sure you don't have any of those lying around in your project.

You can now import these elements together with everything else. For example if you imported the `ecs-utils` lib via `import * as utils from ‘@dcl/ecs-scene-utils'`, you can access _TriggerBoxShape_ as `utils.TriggerBoxShape` and _InterpolationType_ as `utils.InterpolationType`.

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

#### Dialog object NPC only

The `Dialog` object was available both in the `ui-utils` library and the `npc-utils` library with the exact same list of features. To avoid this redundancy, it's now only available as part of the `npc-utils` library, with all the same syntax.
