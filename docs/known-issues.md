# Known Issues

This is a high-level overview of the currently unresolved issues. PhotoPrism follows a zero-bug policy, which means that we do our best to fix any issues we learn about. However, sometimes this is not immediately possible, for example due to dependencies on third-party libraries, other apps, or because it is a specific use case that we do not support at this time.

!!! tldr ""
    In order to improve readability and reduce maintenance, minor issues and those we plan to fix in the short term are not listed here. You can browse all reported bugs, ideas, and planned improvements on [GitHub Issues](https://github.com/photoprism/photoprism/issues).

## Self-Hosted Setup

### Shared Domain

Installation in a sub-directory on a shared domain is generally possible if you run your instance behind a [reverse proxy](getting-started/proxies/traefik.md). At the moment, this method is experimental. We do not recommend it because technical expertise is required and a number of specific issues still need to be addressed:

- [Hosting: Resolve known issues when installing in a sub-directory on a shared domain #2391](https://github.com/photoprism/photoprism/issues/2391)

### Nested Storage Folder

It is not supported to configure the *storage* folder to be inside the *originals* folder unless the name starts with a `.` to indicate that it is hidden. Otherwise, this will result in increasingly longer indexing times and high disk usage, since such nesting causes a loop by re-indexing previously created thumbnails and sidecar files:

- [Config: Prevent nesting of "storage" folder inside "originals" unless hidden #1642](https://github.com/photoprism/photoprism/issues/1642)

In practice, this is not a problem as long as you [follow our documentation](getting-started/docker-compose.md#photoprismstorage) and [examples](https://dl.photoprism.app/docker/). We intend to automatically detect, report, and prevent such issues in the future. Due to the flexible mapping of folders within a Docker container to host directories, this has not yet been implemented.

### Symbolic Links

Symbolic links to directories within the *originals* folder are supported if they are accessible from the environment your instance is running in. However, please note that file links are not supported at this time:

- [Support indexing of symlinked files #1049](https://github.com/photoprism/photoprism/issues/1049)

It is also not supported to mount a symbolic link as *storage* folder, or to use links inside the *storage* folder.

### Failed Imports with Network Storage and Docker

When network-based storage (e.g. SMB/CIFS) is combined with running PhotoPrism under Docker, the originals and import directories should be mounted to the container as a single mount point, rather than as two separate mounts. If this is not done, you may observe 'empty file' errors when the import runs, due to caching of zero-length inode data after the files are moved into the library.

- [Photos fail to index after import, empty file error](https://github.com/photoprism/photoprism/issues/2516)

## User Authentication

### Session Invalidation

User authentication sessions expire automatically after 7 days. In some cases, you may want to ensure that all sessions expire immediately to require re-login, for example, after a password change if the previously used password is known to people who should no longer be able to access your instance:

- [Security: Changing password doesn't invalidate existing auth tokens #1935](https://github.com/photoprism/photoprism/issues/1935)

This is a known limitation with the current implementation. One of the reasons for this trade-off is that the account may be used by background processes that are synchronizing files. Invalidating the session immediately could then lead to data loss, which we consider unacceptable. At the very least, immediate invalidation would require more complex client logic and additional documentation to help developers implement it correctly.

While our team is working on a new implementation as part of full multi-user support, you can manually delete `storage/config/sessions.json` and then restart your instance to revoke all sessions and require re-login.

## File Compatibility

### JPEG: Bad RST Marker

This error can occur when decoding JPEG images that contain consecutive 0xFF bytes, e.g. images that have a "glich" (like a few lines of missing image information at the end) or were created with software that inserts them for padding, although this is based on an edge case of the specification and rather uncommon:

- [Bug: (invalid JPEG format: bad RST marker) #1673](https://github.com/photoprism/photoprism/issues/1673)
- [image/jpeg: "bad RST marker" error when decoding #40130](https://github.com/golang/go/issues/40130)
- [JPEG: Automatically check and repair broken/invalid images #2463](https://github.com/photoprism/photoprism/issues/2463)

## RAW Converters

### JPEG Size Limit

RawTherapee cannot be used to convert RAWs if you want to limit the size of JPEGs. In general, when converting RAW files, the size of the generated JPEG images can be limited using the environment variable `PHOTOPRISM_JPEG_SIZE` or the CLI parameter `--jpeg-size`. However, this does not work with RawTherapee because, unlike Darktable, it does not support CLI options for limiting JPEG size:

- [RAW: PHOTOPRISM_JPEG_SIZE is ignored when converting RAW with RawTherapee #2446](https://github.com/photoprism/photoprism/issues/2446)

It would also be bad for indexing performance if PhotoPrism downsized the generated file after converting it with RawTherapee. As a result, this option is ignored when converting a RAW file with RawTherapee. Whether RawTherapee is used depends on additional settings such as `PHOTOPRISM_DARKTABLE_BLACKLIST` (default: "dng,cr3") and `PHOTOPRISM_DISABLE_DARKTABLE` (default: "false").

## Reporting Bugs ##

Before reporting a bug, please use our [Troubleshooting Checklists](getting-started/troubleshooting/index.md)
to determine the cause of your problem. If you have a general question, need help, it could be a local configuration
issue, or a misunderstanding in how the software works:

- you are welcome to ask in our [Community Chat](https://link.photoprism.app/chat)
- or post your question in [GitHub Discussions](https://link.photoprism.app/discussions)

When reporting a problem, always include the software versions you are using and [other information about your environment](https://photoprism.app/kb/reporting-bugs)
such as [browser, browser plugins](getting-started/troubleshooting/browsers.md), operating system, storage type,
memory size, and processor.

We kindly ask you not to report bugs via GitHub Issues **unless you are certain to have found a fully reproducible and previously unreported issue** that must be fixed directly in the app.

Note that all issue **subscribers receive an email notification** from GitHub for each new comment, so these should only be used for sharing important information and not for personal discussions or questions.
