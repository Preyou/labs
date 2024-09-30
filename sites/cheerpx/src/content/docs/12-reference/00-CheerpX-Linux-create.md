---
title: CheerpX.Linux.create
description: Create a CheerpX application instance
---

```ts
namespace CheerpX {
	class Linux {
		static async create(options?: {
			mounts?: MountPointConfiguration[];
			networkInterface?: NetworkInterface;
			activityInterface?: ActivityInterface;
			bridgeURL?: string;
		}): Promise<CheerpX.Linux>;
	}
}

interface MountPointConfiguration {
	// Specifies the filesystem type
	// 'ext2' for Linux ext2
	// 'dir' for a hierarchical file system
	// 'devs' for device files
	// 'proc' for process info files.
	type: "ext2" | "dir" | "devs" | "proc";

	// First mount must be "/" (root)
	path: string;
	// Can be one of: overlayDevice / webDevice / dataDevice / idbDevice
	dev: CheerpX.Device;
}

interface NetworkInterface {
	authKey?: string;
	controlUrl?: string;
	loginUrlCb?: (url: string) => void;
	stateUpdateCb?: (state: number) => void;
	netmapUpdateCb?: (map: any) => void;
}

interface ActivityInterface {
	// Callback for CPU state changes:
	//'ready' when CPU is active, 'wait' when idle.
	cpu?: (state: "ready" | "wait") => void;

	// Callback for device state changes:
	//'ready' when device is active, 'wait' when idle.
	dev?: (state: "ready" | "wait") => void;
}
```

## Parameters

- **options (`object`, _optional_)** - Used to configure different settings of the CheerpX Linux environment. This object may include settings for mounts, network interfaces, and other features, structured as `{ option: "value" }`.

## Returns

`CheerpX.Linux.create` returns a [Promise] which is resolved once the CheerpX Linux environment is fully initialized and ready to use.

## Options

A description of each `CheerpX.Linux.create` option with brief examples are given below.

### `mounts`

```ts
mounts?: MountPointConfiguration[];
```

This option configures the filesystems that will be available in the CheerpX environment. Each mount point configures a device and specifies where it should be accessible within the virtual filesystem.

Example:

```js
const cx = await CheerpX.Linux.create({
	mounts: [
		{ type: "ext2", path: "/", dev: overlayDevice },
		{ type: "dir", path: "/app", dev: webDevice },
	],
});
```

### `networkInterface`

```ts
networkInterface?: NetworkInterface;
```

This option configures network settings, which allows CheerpX to communicate over networks.

### `activityInterface`

```ts
activityInterface?: ActivityInterface;
```

This option allows monitoring of CPU and device activity within CheerpX, useful for handling state changes and resource management.

### `bridgeURL`

```ts
bridgeURL?: string;
```

This option specifies the URL of a bridge server for communication between the CheerpX environment and external servers.

Example:

```js
const cx = await CheerpX.Linux.create({
	bridgeURL: "https://yourbridgeurl.com/api",
});
```

### Device Configuration Options for CheerpX

CheerpX supports various types of devices that can be configured as overlayDevice. Here’s how you can create them:

- HttpBytesDevice (bytes): The default choice for loading filesystem images via HTTP. Suitable for most web-hosted files.
- CloudDevice (block): Optimized for use with Cloudflare, enhancing performance and reliability through cloud storage solutions.
- GitHubDevice (split): Ideal for projects integrated with GitHub Actions, allowing direct file loading from GitHub repositories.
- IDBDevice: Provides persistent local storage using the browser's IndexedDB, perfect for applications requiring data retention across sessions.

Example: Creating an Overlay Device

```js
const overlayDevice = await CheerpX.OverlayDevice.create(
	await CheerpX.HttpBytesDevice.create("https://yourserver.com/image.ext2"),
	await CheerpX.IDBDevice.create("block1"),
);
```

### Using Different Device Types in Mounts

CheerpX supports various device types that can be mounted and accessed like filesystems within the CheerpX environment.

#### IDBDevice

`CheerpX.IDBDevice` allows you to create a persistent storage device backed by IndexedDB.

Example:

```js
const idbDevice = await CheerpX.IDBDevice.create("dbName");

const cx = await CheerpX.Linux.create({
	mounts: [{ type: "dir", path: "/files", dev: idbDevice }],
});
```

This setup creates a virtual filesystem at `/files` that is backed by IndexedDB.

#### WebDevice

`CheerpX.WebDevice` allows you to mount a directory from your local server.

Example:

```js
const webDevice = await CheerpX.WebDevice.create("path/to/local/directory");

const cx = await CheerpX.Linux.create({
	mounts: [{ type: "dir", path: "/app", dev: webDevice }],
});
```

This mounts the specified local directory at `/app` in the CheerpX environment.

#### DataDevice

`CheerpX.DataDevice` provides a simple in-memory filesystem.

Example:

```js
const dataDevice = await CheerpX.DataDevice.create();

const cx = await CheerpX.Linux.create({
	mounts: [{ type: "dir", path: "/data", dev: dataDevice }],
});
```

This creates an in-memory filesystem mounted at `/data`.

### `dataDevice.writeFile`

`CheerpX.DataDevice` provides a method to write data to new files within the mounted device. This utility is limited to creating files at the root level of the device.

```js
await dataDevice.writeFile(filename: string, contents: string | Uint8Array): Promise<void>
```

**Parameters**:

- **filename**: A string representing the absolute path to the file, starting with a `/` (e.g., "/filename").
- **contents**: The data to write to the file. Can be either a string or a Uint8Array.

**Returns**:

The method returns a Promise that resolves when the file has been created and written to. It doesn't return any value.

Example:

```js
const dataDevice = await CheerpX.DataDevice.create();
await dataDevice.writeFile("/filename", "contents");
```

Note:

- This is the only way to create files in this device.
- Modifying existing files or creating files in subdirectories is not possible.

[Promise]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise