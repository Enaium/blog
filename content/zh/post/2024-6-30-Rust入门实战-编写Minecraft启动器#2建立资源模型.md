---
layout: post
title: "Rust入门实战 编写Minecraft启动器#2建立资源模型"
date: 2024-06-30T23:33:28+08:00
---

我们需要声明几个结构体来存储游戏的资源信息，之后我们需要将`json`文件解析成这几个结构体，所以我们需要添加`serde`依赖。

```toml
serde = { version = "1.0", features = ["derive"] }
```

资源相关`asset.rs`

```rust
use serde::Deserialize;
use std::collections::HashMap;

#[derive(Deserialize)]
pub struct AssetIndex {
    pub id: String,
    pub sha1: String,
    pub size: u32,
    #[serde(alias = "totalSize")]
    pub total_size: u32,
    pub url: String,
}

#[derive(Deserialize)]
pub struct Index {
    pub objects: HashMap<String, Object>,
}

#[derive(Deserialize)]
pub struct Object {
    pub hash: String,
    pub size: u32,
}
```

游戏本体`version.rs`

```rust
use serde::Deserialize;

use crate::{asset::AssetIndex, library::Library};

pub type Libraries = Vec<Library>;

#[derive(Deserialize)]
pub struct Version {
    #[serde(alias = "assetIndex")]
    pub asset_index: AssetIndex,
    pub downloads: Download,
    pub id: String,
    pub libraries: Libraries,
    #[serde(alias = "mainClass")]
    pub main_class: String,
    #[serde(alias = "releaseTime")]
    pub release_time: String,
    pub time: String,
    #[serde(alias = "type")]
    pub type_: String,
}

#[derive(Deserialize)]
pub struct Download {
    pub client: Client,
}

#[derive(Deserialize)]
pub struct Client {
    pub sha1: String,
    pub size: u32,
    pub url: String,
}
```

游戏依赖库`library.rs`

```rust
use serde::Deserialize;

#[derive(Deserialize)]
pub struct Library {
    pub downloads: Download,
    pub name: String,
    pub rules: Option<Vec<Rule>>,
}

#[derive(Deserialize)]
pub struct Rule {
    pub action: String,
    pub os: Os,
}

#[derive(Deserialize)]
pub struct Os {
    pub name: String,
}

#[derive(Deserialize)]
pub struct Download {
    pub artifact: Artifact,
}

#[derive(Deserialize)]
pub struct Artifact {
    pub path: String,
    pub sha1: String,
    pub size: i32,
    pub url: String,
}
```

还有版本清单`version_manifest.rs`

```rust
use serde::Deserialize;

#[derive(Deserialize)]
pub struct VersionManifest {
    pub latest: Latest,
    pub versions: Vec<Version>,
}

#[derive(Deserialize)]
pub struct Latest {
    pub release: String,
    pub snapshot: String,
}

#[derive(Deserialize)]
pub struct Version {
    pub id: String,
    #[serde(alias = "type")]
    pub type_: String,
    pub url: String,
    pub time: String,
    #[serde(alias = "releaseTime")]
    pub release_time: String,
}
```

最后我们把这几个模块导入到`lib.rs`中。

```rust
pub mod asset;
pub mod library;
pub mod version;
pub mod version_manifest;
```

[项目地址](https://github.com/Enaium/teaching-rust-minecraft-client-launch)
