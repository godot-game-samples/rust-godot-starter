# Godot 4 & Rust

<img width="600" src="https://github.com/godot-game-samples/rust-godot-starter/blob/main/assets/screenshot/image_1.png">

To use Rust with Godot 4, use godot-rust.

**GitHub**

[godot-rust](https://github.com/godot-rust/gdext)

**Crate**

[godot](https://crates.io/crates/godot)

First, create a “rust_godot_project” directory so that Godot's project and Rust's project directories are lined up in parallel, as per the official documentation.

```
rust_godot_project
├── godot
└── rust
```

Generate Rust project by cargo new in “rust_godot_project” directory.

```
[lib]
crate-type = ["cdylib"]

[dependencies]
godot = "0.3.1" 
```

godot crate can be specified in the git repository

```
[dependencies]
godot = { git = "https://github.com/godot-rust/gdext.git", branch="master" }
```

Modify lib.rs as follows

```
mod player;
use godot::prelude::*;

struct RustExtension;

#[gdextension]
unsafe impl ExtensionLibrary for RustExtension {}
```

You can use the gdext API by putting godot crate's prelude module in scope.

RustExtension is an arbitrary name, and since we generated the project with “rust” this time, we use “RustExtension”.

Create player.rs.

**player.rs**

```
use godot::prelude::*;
use godot::classes::{ISprite2D, Sprite2D};

#[derive(GodotClass)]
#[class(base=Sprite2D)]
struct Player {
    speed: f64,
    angular_speed: f64,

    base: Base<Sprite2D>
}

#[godot_api]
impl ISprite2D for Player {
    fn init(base: Base<Sprite2D>) -> Self {
        godot_print!("Hello, world!");

        Self {
            speed: 400.0,
            angular_speed: std::f64::consts::PI,
            base,
        }
    }

    fn physics_process(&mut self, delta: f64) {
        let radians = (self.angular_speed * delta) as f32;
        self.base_mut().rotate(radians);
    }
}
```

Create Player.rs based on the official documentation.

This Player structure will be used by Godot.

Once this is done, perform a cargo build.

```
cargo build
```

In this case, an error may occur if rustc is old.

Be aware that the minimum version of the Rust compiler is rustc 1.87, which requires a relatively new rustc. Update with the following version.

```
rustup update stable
```

Version Confirmation

```
rustc --version
rustc 1.87.0 (17067e9ac 2025-05-09)
```

Now we will work on the Godot side.

### Project Creation with Godot

Start Godot and create a project named godot.

The project path will be in the “rust_godot_project” directory created earlier.

<img width="300" src="https://github.com/godot-game-samples/rust-godot-starter/blob/main/assets/screenshot/image_2.png">

Create a new 2D scene by pressing the 2D scene button.

<img width="300" src="https://github.com/godot-game-samples/rust-godot-starter/blob/main/assets/screenshot/image_3.png">

Rename it to "Main.

<img width="300" src="https://github.com/godot-game-samples/rust-godot-starter/blob/main/assets/screenshot/image_4.png">

Create and place the “.gdextension” file in the “godot” project, which is the key to connecting Godot and Rust.

This time, create the file as “rust.gdextension”.

**rust.gdextension**

```
[configuration]
entry_symbol = "gdext_rust_init"
compatibility_minimum = 4.1
reloadable = true

[libraries]
linux.debug.x86_64 =     "res://../rust/target/debug/librust.so"
linux.release.x86_64 =   "res://../rust/target/release/librust.so"
windows.debug.x86_64 =   "res://../rust/target/debug/rust.dll"
windows.release.x86_64 = "res://../rust/target/release/rust.dll"
macos.debug =            "res://../rust/target/debug/librust.dylib"
macos.release =          "res://../rust/target/release/librust.dylib"
macos.debug.arm64 =      "res://../rust/target/debug/librust.dylib"
macos.release.arm64 =    "res://../rust/target/release/librust.dylib"
```

The libraries section must be updated to match the path to the dynamic Rust library.

The path should be written for each platform, but can be limited if only mac is needed.

The file name will be in the form “lib{YourCrate}.dylib”, or in this case “librust.dylib” since we are generating it for the “rust” project.


Now we can add the “Player” struct we just created on the Rust side as a node, and add a Sprite2D “Player” node as a child node of the Main scene.

<img width="300" src="https://github.com/godot-game-samples/rust-godot-starter/blob/main/assets/screenshot/image_5.png">

Adapt “icon.svg” from the file system to the Sprite2D texture.

<img width="300" src="https://github.com/godot-game-samples/rust-godot-starter/blob/main/assets/screenshot/image_6.png">

Move it roughly to the middle.

<img width="400" src="https://github.com/godot-game-samples/rust-godot-starter/blob/main/assets/screenshot/image_7.png">

When godot is built, the icon spins around. It works well with Rust.

<img width="400" src="https://github.com/godot-game-samples/rust-godot-starter/blob/main/assets/screenshot/image_8.png">

### Rotate further according to the tutorial.

Further, follow the tutorial to orbit the icon.

Modify Rust's physics_process as follows

```
fn physics_process(&mut self, delta: f64) {        
    let radians = (self.angular_speed * delta) as f32;
    self.base_mut().rotate(radians);

    let rotation = self.base().get_rotation();
    let velocity = Vector2::UP.rotated(rotation) * self.speed as f32;
    self.base_mut().translate(velocity * delta as f32);
}
```

After the modification, the build of Godot still does not reflect the modification.

You have to do a build for each modification on the Rust side.

```
cargo build
```

Again, build and check Godot. This time it worked.

<img width="400" src="https://github.com/godot-game-samples/rust-godot-starter/blob/main/assets/screenshot/image_9.png">

## Article

Here's an article with more details.

https://www.webcyou.com/?p=12258

## Author

**Daisuke Takayama**

-   [@webcyou](https://twitter.com/webcyou)
-   [@panicdragon](https://twitter.com/panicdragon)
-   <https://github.com/webcyou>
-   <https://github.com/webcyou-org>
-   <https://github.com/panicdragon>
-   <https://www.webcyou.com/>
