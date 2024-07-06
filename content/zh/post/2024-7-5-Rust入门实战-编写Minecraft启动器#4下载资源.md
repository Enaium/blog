---
layout: post
title: "Rust入门实战 编写Minecraft启动器#4下载资源"
date: 2024-07-05T19:58:57+08:00
---

首先我们需要添加几个依赖。

```toml
model = { path = "../model" }
parse = { path = "../parse" }
reqwest = { version = "0.12", features = ["blocking", "json"] }
file-hashing = { version = "0.1" }
sha1 = { version = "0.10" }
```

`reqwest`用于发送请求，`file-hashing`用于计算文件的`hash`，`sha1`用于计算`sha1`。

之后我们需要添加下载的`trait`。

```rust
pub trait Download {
    fn download(&self, game_dir: &Path) -> Result<(), Box<dyn std::error::Error>>;
}
```

接着我们需要使用`Client::builder()`来创建一个`Client`，因为默认的`get`方法会用有个超时时间，而我们需要设置超时时间为无限。

```rust
pub fn get<T: reqwest::IntoUrl>(url: T) -> reqwest::Result<reqwest::blocking::Response> {
    reqwest::blocking::Client::builder()
        .timeout(None)
        .build()?
        .get(url)
        .send()
}
```

最后我们需要创建一个计算文件`hash`的函数。

```rust
pub fn sha1<P: AsRef<Path>>(path: P) -> Result<String, std::io::Error> {
    let mut hasher = Sha1::new();
    file_hashing::get_hash_file(path, &mut hasher)
}
```

之后需要出创建`asset.rs`、`library.rs`、`version.rs`文件，分别对应下载资源、下载库、下载游戏版本。

`asset.rs`

```rust
use std::{fs, path::Path};

use model::asset::*;
use parse::Parse;

use crate::{get, Download};

impl Download for AssetIndex {
    fn download(&self, game_dir: &Path) -> Result<(), Box<dyn std::error::Error>> {
        println!("Downloading asset index:{}", self.id);

        let indexes_dir = &game_dir.join("assets").join("indexes");

        if !indexes_dir.exists() {
            std::fs::create_dir_all(indexes_dir)?;
        }

        let path = &indexes_dir.join(&format!("{}.json", self.id));

        std::fs::File::create(path)?;

        let url = &self.url;
        let text = &get(url)?.text()?;

        std::fs::write(path, text)?;

        let index = Index::parse(text)?;

        let objects_dir = &game_dir.join("assets").join("objects");

        if !objects_dir.exists() {
            std::fs::create_dir_all(objects_dir)?;
        }

        for (_, value) in index.objects {
            let hash = &value.hash;
            let hash_first_two = &hash.chars().take(2).collect::<String>();

            let first_two_dir = &objects_dir.join(hash_first_two);

            if !first_two_dir.exists() {
                std::fs::create_dir_all(first_two_dir)?;
            }

            let path = &first_two_dir.join(hash);

            if path.exists() {
                if crate::sha1(path)?.eq(hash) {
                    continue;
                } else {
                    std::fs::remove_file(path)?;
                }
            }

            std::fs::File::create(path)?;

            let url = format!(
                "https://resources.download.minecraft.net/{}/{}",
                hash_first_two, hash
            );

            println!("Downloading:{}", url);

            let bytes = get(&url)?.bytes()?;
            fs::write(path, bytes)?;
        }
        Ok(())
    }
}

#[cfg(test)]
mod tests {

    use super::*;

    #[test]
    fn test_asset_index() {
        let asset_index = model::asset::AssetIndex {
            id: "17".to_string(),
            sha1: "fab15439bdef669e389e25e815eee8f1b2aa915e".to_string(),
            size: 447033,
            total_size: 799252591,
            url: "https://piston-meta.mojang.com/v1/packages/fab15439bdef669e389e25e815eee8f1b2aa915e/17.json".to_string(),
        };

        let download_path = &std::env::temp_dir().join("rust-minecraft-client-launch");
        std::fs::create_dir_all(download_path).unwrap_or_else(|err| panic!("{:?}", err));

        if let Err(err) = asset_index.download(download_path) {
            panic!("{:?}", err);
        }
    }
}
```

`library.rs`

```rust
use std::path::Path;

use model::{library, version::Libraries};

use crate::{Download, LibraryAllowed};

impl LibraryAllowed for library::Library {
    fn allowed(&self) -> bool {
        let mut allowed = true;

        if self.rules.is_some() {
            for rule in self.rules.as_ref().unwrap() {
                if rule.os.name == "osx" && !cfg!(target_os = "macos") {
                    allowed = false;
                    break;
                } else if rule.os.name == "linux" && !cfg!(target_os = "linux") {
                    allowed = false;
                    break;
                } else if rule.os.name == "windows" && !cfg!(target_os = "windows") {
                    allowed = false;
                    break;
                }
            }
        }

        if self.name.contains("natives") {
            if self.name.contains("x86") && !cfg!(target_arch = "x86") {
                allowed = false;
            } else if self.name.contains("arm64") && !cfg!(target_arch = "aarch64") {
                allowed = false;
            } else if !cfg!(target_arch = "x86_64") {
                allowed = false;
            }
        }

        allowed
    }
}

impl Download for Libraries {
    fn download(&self, game_dir: &Path) -> Result<(), Box<dyn std::error::Error>> {
        println!("Downloading libraries");

        let libraries_dir = &game_dir.join("libraries");

        if !libraries_dir.exists() {
            std::fs::create_dir_all(libraries_dir)?;
        }

        for library in self {
            if !library.allowed() {
                continue;
            }

            let library_file = &library.downloads.artifact.path;

            let library_path = &libraries_dir.join(library_file);

            if !library_path.parent().unwrap().exists() {
                std::fs::create_dir_all(library_path.parent().unwrap())?;
            }

            if library_path.exists() {
                if crate::sha1(library_path)? == library.downloads.artifact.sha1 {
                    continue;
                } else {
                    std::fs::remove_file(library_path)?;
                }
            }

            std::fs::File::create(&library_path)?;

            let url = &library.downloads.artifact.url;

            println!("Downloading: {}", url);

            let bytes = crate::get(url)?.bytes()?;

            std::fs::write(library_path, bytes)?;
        }

        Ok(())
    }
}

#[cfg(test)]
mod tests {
    use super::*;
    use model::version::Version;

    #[test]
    fn test_download() {
        let game = reqwest::blocking::get("https://piston-meta.mojang.com/v1/packages/177e49d3233cb6eac42f0495c0a48e719870c2ae/1.21.json")
            .unwrap()
            .json::<Version>()
            .unwrap();

        let download_path = &std::env::temp_dir().join("rust-minecraft-client-launch");
        std::fs::create_dir_all(download_path).unwrap_or_else(|err| panic!("{:?}", err));

        if let Err(err) = game.libraries.download(download_path) {
            panic!("{:?}", err);
        }
    }
}
```

这里我们需要添加一个`trait`，用于判断库是否允许下载。

```rust
pub trait LibraryAllowed {
    fn allowed(&self) -> bool;
}
```

`version.rs`

```rust
use std::path::Path;

use model::version_manifest::Version;

use crate::{get, sha1, Download};

impl Download for Version {
    fn download(&self, game_dir: &Path) -> Result<(), Box<dyn std::error::Error>> {
        let game = get(&self.url)
            .unwrap()
            .json::<model::version::Version>()
            .unwrap();

        let versions_dir = &game_dir.join(game_dir).join("versions").join(&self.id);

        if !versions_dir.exists() {
            std::fs::create_dir_all(versions_dir)?;
        }

        game.libraries.download(game_dir)?;
        game.asset_index.download(game_dir)?;

        let version_config = &game_dir
            .join("versions")
            .join(&self.id)
            .join(&format!("{}.json", &self.id));

        if version_config.exists() {
            std::fs::remove_file(version_config).unwrap();
        }

        std::fs::File::create(version_config).unwrap();
        std::fs::write(version_config, get(&self.url).unwrap().bytes().unwrap()).unwrap();

        let path = &versions_dir
            .join(versions_dir)
            .join(&format!("{}.jar", &self.id));

        if path.exists() {
            if sha1(path)? == game.downloads.client.sha1 {
                return Ok(());
            } else {
                std::fs::remove_file(path)?;
            }
        }

        std::fs::File::create(path)?;

        let bytes = crate::get(&game.downloads.client.url)?.bytes()?;

        std::fs::write(path, bytes)?;

        Ok(())
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_download() {
        let version = Version {
            id: "1.21".to_string(),
            type_: "release".to_string(),
            url: "https://piston-meta.mojang.com/v1/packages/177e49d3233cb6eac42f0495c0a48e719870c2ae/1.21.json".to_string(),
            time : "2024-06-13T08:32:38+00:00".to_string(),
            release_time : "2024-06-13T08:24:03+00:00".to_string(),
        };

        let download_path = &std::env::temp_dir().join("rust-minecraft-client-launch");
        std::fs::create_dir_all(download_path).unwrap_or_else(|err| panic!("{:?}", err));

        if let Err(err) = version.download(download_path) {
            panic!("{:?}", err);
        }
    }
}
```

好了，现在我们可以测试下载资源了。