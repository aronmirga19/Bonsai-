<h1 align="center" style="font-family:Papyrus; font-size:4em;"> Bonsai 盆栽 </h1>
<p align="center">
  <img src="https://github.com/Sollimann/bonsai/blob/main/docs/resources/gifs/bonsai.gif" width="350" ">
</p>

<p align="center">
    <em>Rust implementation of Behavior Trees</em>
</p>

<!-- [![codecov](https://codecov.io/gh/Sollimann/CleanIt/branch/main/graph/badge.svg?token=EY3JRZN71M)](https://codecov.io/gh/Sollimann/CleanIt) -->
<!-- [![version](https://img.shields.io/badge/version-1.0.0-blue)](https://GitHub.com/Sollimann/CleanIt/releases/) -->
[![Build Status](https://github.com/Sollimann/bonsai/workflows/rust-ci/badge.svg)](https://github.com/Sollimann/bonsai/actions)
[![minimum rustc 1.60](https://img.shields.io/badge/rustc-1.60+-blue.svg)](https://rust-lang.github.io/rfcs/2495-min-rust-version.html)
[![Maintenance](https://img.shields.io/badge/Maintained%3F-yes-green.svg)](https://GitHub.com/Sollimann/bonsai/graphs/commit-activity)
[![GitHub pull-requests](https://img.shields.io/github/issues-pr/Sollimann/bonsai.svg)](https://GitHub.com/Sollimann/bonsai/pulls)
[![GitHub pull-requests closed](https://img.shields.io/github/issues-pr-closed/Sollimann/bonsai.svg)](https://GitHub.com/Sollimann/bonsai/pulls)
![ViewCount](https://views.whatilearened.today/views/github/Sollimann/bonsai.svg)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)



## What is a Behavior Tree?

A _Behavior Tree_ (BT) is a data structure in which we can set the rules of how certain _behavior's_ can occur, and the order in which they would execute. BTs are a very efficient way of creating complex systems that are both modular and reactive. These properties are crucial in many applications, which has led to the spread of BT from computer game programming to many branches of AI and Robotics.

<p align="center">
  <img src="https://github.com/Sollimann/bonsai/blob/readme-example/docs/resources/images/npc_bt.png" width="600" ">
</p>

```rust
use std::{collections::HashMap, thread::sleep, time::Duration};

use bonsai::{
    Behavior::{Action, Select, Sequence},
    Event, Timer, UpdateArgs, BT,
};

type Damage = u32;
type Distance = f64;
type Time = f64;

#[derive(serde::Deserialize, serde::Serialize, Clone, Debug, PartialEq)]
enum EnemyNPC {
    /// Run
    Run,
    /// Cover
    GetInCover,
    /// Blind fire for some time
    BlindFire(Time, Damage),
    /// When player is close -> melee attack
    MeleeAttack(Damage),
    /// When player is far away -> fire weapon
    FireWeapon(Damage),
    /// Return true if player within striking distance
    PlayerIsClose(Distance),
}

fn game_tick(timer: &mut Timer, bt: &mut BT<EnemyNPC, String, serde_json::Value>) {
    // time since last invovation of bt
    let dt = timer.get_dt();

    // proceed to next iteration in event loop
    let e: Event = UpdateArgs { dt }.into();

    #[rustfmt::skip]
    bt.state.tick(&e,&mut |args: bonsai::ActionArgs<Event, EnemyNPC>| {
        match *args.action {
            EnemyNPC::Run => todo!(),
            EnemyNPC::GetInCover => todo!(),
            EnemyNPC::BlindFire(_, _) => todo!(),
            EnemyNPC::MeleeAttack(_) => todo!(),
            EnemyNPC::FireWeapon(_) => todo!(),
            EnemyNPC::PlayerIsClose(_) => todo!(),
        }
    });
}

fn main() {
    use crate::EnemyNPC::{BlindFire, FireWeapon, GetInCover, MeleeAttack, Run};

    // define blackboard (even though we're not using it)
    let blackboard: HashMap<String, serde_json::Value> = HashMap::new();

    // create ai behavior
    let run = Action(Run);
    let get_in_cover = Action(GetInCover);

    let run_cover = Sequence(vec![run, get_in_cover, Action(BlindFire(2.0, 50))]);
    let player_close = Select(vec![Action(MeleeAttack(100)), Action(FireWeapon(50))]);
    let under_fire_behavior = Select(vec![run_cover, player_close]);

    let bt_serialized = serde_json::to_string_pretty(&under_fire_behavior).unwrap();
    println!("creating bt: \n {} \n", bt_serialized);
    let mut bt = BT::new(under_fire_behavior, blackboard);

    // create a monotonic timer
    let mut timer = Timer::init_time();

    loop {
        // decide bt frequency by sleeping the loop
        sleep(Duration::new(0, 0.1e+9 as u32));

        // tick the bt
        game_tick(&mut timer, &mut bt);
    }
}
```

## Contents

* [Concepts](docs/concepts/README.md)
* [Development Guide](DEVELOPMENT.md)
* [Kanban Board](https://github.com/Sollimann/b3/projects/1)
