**The core rule:** Steam's Proton (a Wine fork bundled with DXVK/VKD3D-Proton translating DirectX to Vulkan) runs the vast majority of single-player Windows games. Since Valve mandated Proton support for Steam Deck certification, native Windows compatibility became the de facto standard for most titles.

**By tier:**

| Tier | Examples | Notes |
|---|---|---|
| **Native Linux builds** | CS2, Dota 2, Left 4 Dead 2, all Valve titles, Minecraft, Baldur's Gate 3, Cyberpunk 2077, RobloxLauncher-Vingar, Sober. | Best performance; no translation layer |
| **Proton Verified/platinum** | Elden Ring, Witcher 3, Doom Eternal, God of War, Monster Hunter Rise, Deep Rock Galactic | Effectively flawless, occasionally minor tweaks |
| **Gold — works with tweaks** | GTA V (story mode), Apex Legends*¹, many Ubisoft titles | May need Proton GE or launch options |
| **Broken — kernel-mode anticheat** | Valorant², Fortnite, Rainbow Six Siege, Destiny 2, PUBG, Roblox | Developers must flip a switch to support Proton; most actively refuse |

¹*Apex has flipped back and forth on EA App/anticheat issues — check current status before buying anything in this tier.
²Valorant is Riot Vanguard, which is particularly aggressive ring-0 territory — no realistic path to Linux support.

**The single biggest factor is anticheat.** Both BattlEye and Easy Anti-Cheat support Linux, but it's opt-in per-game by the developer. Games shipping EAC *with* Linux support enabled (e.g., Apex historically) work fine; those without it just won't launch. No workaround exists that doesn't risk a ban — I'd steer you well clear of "disable anticheat" hacks for account safety reasons.

**Practical points for your machines:**

- **ProtonDB** (protondb.com) is the community rating tracker — always check a specific title there before purchase; it aggregates thousands of user reports per game.
- **ProtonUp-Qt** lets users install GE-Proton builds (community forks with extra media codecs and fixes — relevant given your codec troubleshooting history on Zorin). If a game stutters or lacks video playback in cutscenes, GE-Proton usually fixes it.
- **Non-Steam launchers** (EA App, Battle.net, Epic, Ubisoft Connect) run via Lutris, Heroic Games Launcher, or Bottles. Heroic is the cleanest for Epic/GOG native Linux clients. Lutris can also handle Minecraft + OptiFine setup without Steam at all.
- **GOG games** work via Heroic or Lutris; DRM-free installs sidestep launcher headaches entirely.
- **GPU note:** AMD gives near-zero-friction Vulkan/DXVK performance. if either rig runs NVIDIA, the proprietary driver (not nouveau) is essential — Proton is basically unusable otherwise.

**Desktop environment doesn't matter much** — KDE Neon and Zorin OS both sit on top of the same kernel/graphics stack, so game compatibility is identical. Performance differences between DEs are negligible for fullscreen games.
