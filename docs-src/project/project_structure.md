# Common Project Structure

```
my_game/
├── project.godot
├── .godot/ (auto-generated, gitignore this)
│
├── autoload/ (or globals/, singletons/)
│   ├── game_manager.gd
│   ├── save_manager.gd
│   ├── audio_manager.gd
│   ├── economy_manager.gd
│   └── adventurer_manager.gd
│
├── scenes/
│   ├── main/
│   │   ├── boot.tscn
│   │   ├── main_menu.tscn
│   │   └── game.tscn
│   ├── characters/
│   │   ├── player/
│   │   │   ├── player.tscn
│   │   │   └── player.gd
│   │   └── enemies/
│   │       ├── goblin.tscn
│   │       └── goblin.gd
│   ├── buildings/
│   │   ├── shop.tscn
│   │   ├── blacksmith.tscn
│   │   └── herbalist.tscn
│   └── ui/
│       ├── hud.tscn
│       ├── inventory.tscn
│       └── pause_menu.tscn
│
├── scripts/
│   ├── classes/ (data classes)
│   │   ├── adventurer_data.gd
│   │   ├── item_data.gd
│   │   └── quest_data.gd
│   └── utils/
│       ├── constants.gd
│       └── helpers.gd
│
├── assets/
│   ├── sprites/
│   │   ├── characters/
│   │   ├── items/
│   │   └── ui/
│   ├── audio/
│   │   ├── music/
│   │   └── sfx/
│   └── fonts/
│
└── resources/ (.tres files)
    ├── items/
    │   ├── sword.tres
    │   └── potion.tres
    ├── themes/
    │   └── main_theme.tres
    └── data/
        └── game_config.tres
```
