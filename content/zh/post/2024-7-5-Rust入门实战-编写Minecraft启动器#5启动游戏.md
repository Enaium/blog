---
title: "Rust入门实战 编写Minecraft启动器#5启动游戏"
date: 2024-07-05T20:19:35+08:00
---

好了，我们已经完成了所有的准备工作，现在我们可以开始编写启动游戏的代码了。

首先我们需要添加几个依赖。

```toml
model = { path = "../model" }
parse = { path = "../parse" }
download = { path = "../download" }
clap = { version = "4.5" }
zip = "2.1"
```

`clap`用于解析命令行参数，`zip`用于解压文件。

首先创建一个`cli`函数用于构建我们的命令行。

```rust
fn cli() -> Command {
    Command::new("rmcl")
        .about("A Minecraft launcher written in Rust")
        .version("0.1.0")
        .author("Enaium")
        .subcommand_required(true)
        .arg_required_else_help(true)
        .allow_external_subcommands(true)
        .subcommand(
            Command::new("search")
                .about("Search Game")
                .arg(arg!([VERSION] "Game version"))
                .arg(
                    arg!(-t --type <TYPE> "Game type")
                        .value_parser(["release", "snapshot", "old_beta", "old_alpha"])
                        .require_equals(true)
                        .default_value("release")
                        .default_missing_value("release"),
                ),
        )
        .subcommand(
            Command::new("download")
                .about("Download Game")
                .arg(arg!(<VERSION> "Game Version"))
                .arg_required_else_help(true),
        )
        .subcommand(
            Command::new("launch")
                .about("Launch Game")
                .arg(arg!(<VERSION> "Game Version"))
                .arg_required_else_help(true),
        )
}
```

接着创建一个`get_version_manifest`函数用于获取游戏的所有版本清单。

```rust
fn get_version_manifest() -> model::version_manifest::VersionManifest {
    return get("https://launchermeta.mojang.com/mc/game/version_manifest.json")
        .unwrap()
        .json::<version_manifest::VersionManifest>()
        .unwrap();
}
```

之后编写每个子命令的处理函数。

```rust
fn search(sub_matches: &clap::ArgMatches) {
    let version = sub_matches.get_one::<String>("VERSION");
    let type_ = sub_matches.get_one::<String>("type").unwrap();
    let versions = get_version_manifest().versions;

    let versions = versions.iter().filter(|v| {
        (if version.is_some() {
            v.id.contains(version.unwrap())
        } else {
            true
        }) && v.type_.eq(type_)
    });
    for version in versions {
        println!("Version:{}", version.id);
    }
}
```

```rust
fn download(sub_matches: &clap::ArgMatches) {
    let game_dir = std::env::current_dir().unwrap().join(".minecraft");
    let version = sub_matches.get_one::<String>("VERSION").unwrap();
    let versions = get_version_manifest().versions;

    if let Some(version) = versions.iter().find(|v| v.id.eq(version)) {
        version.download(&game_dir).unwrap_or_else(|err| {
            panic!("Version download error:{:?}", err);
        });
    } else {
        println!("Version:{} not found", version);
    }
}
```

```rust
fn launch(sub_matches: &clap::ArgMatches) {
    let game_dir = std::env::current_dir().unwrap().join(".minecraft");
    let libraries_dir = game_dir.join("libraries");
    let assets_dir = game_dir.join("assets");
    let version = sub_matches.get_one::<String>("VERSION").unwrap();
    let version_dir = game_dir.join("versions").join(version);
    let natives_dir = version_dir.join("natives");
    let config_path = version_dir.join(format!("{}.json", version));
    let version_path = version_dir.join(format!("{}.jar", version));

    if !version_path.exists() || !config_path.exists() {
        println!("Version:{} not found", version_path.display());
        return;
    }

    let version = &Version::parse(&std::fs::read_to_string(&config_path).unwrap()).unwrap();

    for library in &version.libraries {
        if library.allowed() && library.name.contains("natives") {
            extract_jar(
                &libraries_dir.join(&library.downloads.artifact.path),
                &natives_dir,
            );
        }
    }

    let classpath = format!(
        "{}{}",
        &version
            .libraries
            .iter()
            .map(|library| {
                format!(
                    "{}{}",
                    libraries_dir
                        .join(&library.downloads.artifact.path)
                        .display(),
                    if cfg!(windows) { ";" } else { ":" }
                )
            })
            .collect::<String>(),
        version_path.display()
    );

    std::process::Command::new("java")
        .current_dir(&game_dir)
        .arg(format!("-Djava.library.path={}", natives_dir.display()))
        .arg("-Dminecraft.launcher.brand=rmcl")
        .arg("-cp")
        .arg(classpath)
        .arg(&version.main_class)
        .arg("--username")
        .arg("Enaium")
        .arg("--version")
        .arg(&version.id)
        .arg("--gameDir")
        .arg(game_dir)
        .arg("--assetsDir")
        .arg(assets_dir)
        .arg("--assetIndex")
        .arg(&version.asset_index.id)
        .arg("--accessToken")
        .arg("0")
        .arg("--versionType")
        .arg("RMCL 0.1.0")
        .status()
        .unwrap();
}
```

解压`natives`文件。

```rust
fn extract_jar(jar: &Path, dir: &Path) {
    if !dir.exists() {
        std::fs::create_dir_all(dir).unwrap();
    }

    let mut archive = zip::ZipArchive::new(std::fs::File::open(jar).unwrap()).unwrap();
    for i in 0..archive.len() {
        let mut entry = archive.by_index(i).unwrap();
        if entry.is_file() && !entry.name().contains("META-INF") {
            let mut name = entry.name();

            if name.contains("/") {
                name = &name[entry.name().rfind('/').unwrap() + 1..];
            }

            let path = dir.join(name);

            if path.exists() {
                std::fs::remove_file(&path).unwrap();
            }

            let mut file = std::fs::File::create(&path).unwrap();

            std::io::copy(&mut entry, &mut file).unwrap();
        }
    }
}
```

最后在`main`函数中调用`cli`函数。

```rust
fn main() {
    let matches = cli().get_matches();

    match matches.subcommand() {
        Some(("search", sub_matches)) => {
            search(sub_matches);
        }
        Some(("download", sub_matches)) => {
            download(sub_matches);
        }
        Some(("launch", sub_matches)) => {
            launch(sub_matches);
        }
        _ => unreachable!(),
    }
}
```

现在我们可以使用`rmcl`命令来搜索、下载、启动游戏了。

![20240705202955](https://s2.loli.net/2024/07/05/Gs3lipmVvNDuzX7.png)

[项目地址](https://github.com/Enaium/teaching-rust-minecraft-client-launch)
