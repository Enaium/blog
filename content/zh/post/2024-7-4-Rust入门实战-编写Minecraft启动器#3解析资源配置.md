---
layout: post
title: "Rust入门实战 编写Minecraft启动器#3解析资源配置"
date: 2024-07-04T23:29:27+08:00
---

在上一篇文章中，我们已经建立了资源模型，接下来我们需要解析游戏的配置文件。

首先我们添加`serde_json`依赖和`model`依赖。

```toml
model = { path = "../model" }
serde_json = "1.0"
```

之后我们在`lib.rs`中添加解析的`trait`。

```rust
pub trait Parse<T>: Sized {
    type Error;
    fn parse(value: T) -> Result<Self, Self::Error>;
}
```

之后将所有的`model`都实现这个`trait`，并测试它们。这里其实只用将需要手动解析的实现这个`trait`，其他的会在我们用`reqwest`下载的时候自动解析。

`asset.rs`

```rust
use model::asset::*;

use crate::Parse;

impl Parse<&str> for AssetIndex {
    type Error = serde_json::Error;

    fn parse(value: &str) -> Result<Self, Self::Error> {
        serde_json::from_str::<AssetIndex>(value)
    }
}

impl Parse<&str> for Index {
    type Error = serde_json::Error;

    fn parse(value: &str) -> Result<Self, Self::Error> {
        serde_json::from_str::<Index>(value)
    }
}

impl Parse<&str> for Object {
    type Error = serde_json::Error;

    fn parse(value: &str) -> Result<Self, Self::Error> {
        serde_json::from_str::<Object>(value)
    }
}

#[cfg(test)]
mod tests {

    use super::*;

    #[test]
    fn test_asset_index() {
        let asset_index = AssetIndex::parse(
        r#"{"id": "17", "sha1": "fab15439bdef669e389e25e815eee8f1b2aa915e", "size": 447033, "totalSize": 799252591, "url": "https://piston-meta.mojang.com/v1/packages/fab15439bdef669e389e25e815eee8f1b2aa915e/17.json"}"#).unwrap_or_else(|err| panic!("{:?}",err));
        assert_eq!("17", asset_index.id);
        assert_eq!("fab15439bdef669e389e25e815eee8f1b2aa915e", asset_index.sha1);
        assert_eq!(447033, asset_index.size);
        assert_eq!(799252591, asset_index.total_size);
        assert_eq!("https://piston-meta.mojang.com/v1/packages/fab15439bdef669e389e25e815eee8f1b2aa915e/17.json", asset_index.url);
    }

    #[test]
    fn test_index() {
        let index = Index::parse(r#"{"objects": {"icons/icon_128x128.png": {"hash": "b62ca8ec10d07e6bf5ac8dae0c8c1d2e6a1e3356", "size": 9101}}}"#)
            .unwrap_or_else(|err| panic!("{:?}",err));

        assert_eq!(1, index.objects.len());
        assert_eq!(
            "b62ca8ec10d07e6bf5ac8dae0c8c1d2e6a1e3356",
            index.objects.get("icons/icon_128x128.png").unwrap().hash
        );
        assert_eq!(
            9101,
            index.objects.get("icons/icon_128x128.png").unwrap().size
        );
    }
}
```

`library.rs`

```rust
use model::library::*;

use crate::Parse;

impl Parse<&str> for Library {
    type Error = serde_json::Error;

    fn parse(value: &str) -> Result<Self, Self::Error> {
        serde_json::from_str::<Library>(value)
    }
}

impl Parse<&str> for Rule {
    type Error = serde_json::Error;

    fn parse(value: &str) -> Result<Self, Self::Error> {
        serde_json::from_str::<Rule>(value)
    }
}

impl Parse<&str> for Os {
    type Error = serde_json::Error;

    fn parse(value: &str) -> Result<Self, Self::Error> {
        serde_json::from_str::<Os>(value)
    }
}

impl Parse<&str> for Download {
    type Error = serde_json::Error;

    fn parse(value: &str) -> Result<Self, Self::Error> {
        serde_json::from_str::<Download>(value)
    }
}

impl Parse<&str> for Artifact {
    type Error = serde_json::Error;

    fn parse(value: &str) -> Result<Self, Self::Error> {
        serde_json::from_str::<Artifact>(value)
    }
}

#[cfg(test)]
mod tests {

    use super::*;

    #[test]
    fn test_library() {
        let library = Library::parse(
                r#"{"downloads": {"artifact": {"path": "ca/weblite/java-objc-bridge/1.1/java-objc-bridge-1.1.jar", "sha1": "1227f9e0666314f9de41477e3ec277e542ed7f7b", "size": 1330045, "url": "https://libraries.minecraft.net/ca/weblite/java-objc-bridge/1.1/java-objc-bridge-1.1.jar"}}, "name": "ca.weblite:java-objc-bridge:1.1", "rules": [{"action": "allow", "os": {"name": "osx"}}]}"#,
            ).unwrap_or_else(|err| panic!("{:?}",err));

        assert_eq!("ca.weblite:java-objc-bridge:1.1", library.name);
        assert_eq!(
            "ca/weblite/java-objc-bridge/1.1/java-objc-bridge-1.1.jar",
            library.downloads.artifact.path
        );
        assert_eq!(
            "1227f9e0666314f9de41477e3ec277e542ed7f7b",
            library.downloads.artifact.sha1
        );
        assert_eq!(1330045, library.downloads.artifact.size);
        assert_eq!(
            "https://libraries.minecraft.net/ca/weblite/java-objc-bridge/1.1/java-objc-bridge-1.1.jar",
            library.downloads.artifact.url
        );
        let rules = &library.rules.unwrap();
        assert_eq!("allow", rules[0].action);
        assert_eq!("osx", rules[0].os.name);
    }
}
```

`version_manifest.rs`

```rust
use model::version_manifest::*;

use crate::Parse;

impl Parse<&str> for VersionManifest {
    type Error = serde_json::Error;

    fn parse(value: &str) -> Result<Self, Self::Error> {
        serde_json::from_str::<VersionManifest>(value)
    }
}

impl Parse<&str> for Latest {
    type Error = serde_json::Error;

    fn parse(value: &str) -> Result<Self, Self::Error> {
        serde_json::from_str::<Latest>(value)
    }
}

impl Parse<&str> for Version {
    type Error = serde_json::Error;

    fn parse(value: &str) -> Result<Self, Self::Error> {
        serde_json::from_str::<Version>(value)
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_version() {
        let version = Version::parse(
            r#"{"id": "1.21", "type": "release", "url": "https://piston-meta.mojang.com/v1/packages/177e49d3233cb6eac42f0495c0a48e719870c2ae/1.21.json", "time": "2024-06-13T08:32:38+00:00", "releaseTime": "2024-06-13T08:24:03+00:00"}"#,
        ).unwrap_or_else(|err| panic!("{:?}",err));

        assert_eq!("1.21", version.id);
        assert_eq!("release", version.type_);
        assert_eq!("https://piston-meta.mojang.com/v1/packages/177e49d3233cb6eac42f0495c0a48e719870c2ae/1.21.json", version.url);
        assert_eq!("2024-06-13T08:32:38+00:00", version.time);
        assert_eq!("2024-06-13T08:24:03+00:00", version.release_time);
    }

    #[test]
    fn test_latest() {
        let latest = Latest::parse(r#"{"release": "1.21", "snapshot": "1.21"}"#)
            .unwrap_or_else(|err| panic!("{:?}", err));
        assert_eq!("1.21", latest.release);
        assert_eq!("1.21", latest.snapshot);
    }

    #[test]
    fn test_version_manifest() {
        let version_manifest =
            VersionManifest::parse(r#"{"latest": {"release": "1.21", "snapshot": "1.21"}, "versions": [{"id": "1.21", "type": "release", "url": "https://piston-meta.mojang.com/v1/packages/177e49d3233cb6eac42f0495c0a48e719870c2ae/1.21.json", "time": "2024-06-13T08:32:38+00:00", "releaseTime": "2024-06-13T08:24:03+00:00"}]}"#).unwrap_or_else(|err| panic!("{:?}", err));
        assert_eq!("1.21", version_manifest.latest.release);
        assert_eq!("1.21", version_manifest.latest.snapshot);
        assert_eq!("1.21", version_manifest.versions[0].id);
        assert_eq!("release", version_manifest.versions[0].type_);
        assert_eq!("https://piston-meta.mojang.com/v1/packages/177e49d3233cb6eac42f0495c0a48e719870c2ae/1.21.json", version_manifest.versions[0].url);
        assert_eq!(
            "2024-06-13T08:32:38+00:00",
            version_manifest.versions[0].time
        );
        assert_eq!(
            "2024-06-13T08:24:03+00:00",
            version_manifest.versions[0].release_time
        );
    }
}
```

`version.rs`

```rust
use crate::Parse;
use model::version::*;

impl Parse<&str> for Version {
    type Error = serde_json::Error;

    fn parse(value: &str) -> Result<Self, Self::Error> {
        serde_json::from_str::<Version>(value)
    }
}

impl Parse<&str> for Download {
    type Error = serde_json::Error;

    fn parse(value: &str) -> Result<Self, Self::Error> {
        serde_json::from_str::<Download>(value)
    }
}

impl Parse<&str> for Client {
    type Error = serde_json::Error;

    fn parse(value: &str) -> Result<Self, Self::Error> {
        serde_json::from_str::<Client>(value)
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_version() {
        let game = Version::parse(
            r#"{"downloads": {"client": {"sha1": "0e9a07b9bb3390602f977073aa12884a4ce12431", "size": 26836080, "url": "https://piston-data.mojang.com/v1/objects/0e9a07b9bb3390602f977073aa12884a4ce12431/client.jar"}}, "id": "1.21", "libraries": [], "mainClass": "net.minecraft.client.main.Main", "releaseTime": "2024-06-13T08:24:03+00:00", "time": "2024-06-13T08:32:38+00:00", "type": "release"}"#,
        ).unwrap_or_else(|err| panic!("{:?}",err));

        let client = &game.downloads.client;
        assert_eq!("0e9a07b9bb3390602f977073aa12884a4ce12431", client.sha1);
        assert_eq!(26836080, client.size);
        assert_eq!("https://piston-data.mojang.com/v1/objects/0e9a07b9bb3390602f977073aa12884a4ce12431/client.jar", client.url);

        assert_eq!("1.21", game.id);
        assert_eq!("net.minecraft.client.main.Main", game.main_class);
        assert_eq!("2024-06-13T08:24:03+00:00", game.release_time);
        assert_eq!("2024-06-13T08:32:38+00:00", game.time);
        assert_eq!("release", game.type_);
    }
}
```

最后我们将这些`tait`导出。

```rust
pub mod asset;
pub mod library;
pub mod version;
pub mod version_manifest;
```