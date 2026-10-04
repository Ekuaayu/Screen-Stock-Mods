# Screen Stocks Master Bot

A community C# trading dashboard and bot for **Screen Stocks Demo**. The current simplified edition uses the game's supported local mod files for market data and trade commands. It includes automated and manual trading, training tools, risk controls, and a custom skin manager.

> [!WARNING]
> **The bot can lose in-game money.** Its training is incomplete, historical/simulated results do not predict future prices, and no strategy can guarantee profit or prevent losses. Automatic trading can submit real trades in your game account. Start in **MANUAL** mode, review the risk settings, and only enable **AUTO** if you accept that risk. This project is a work in progress and is not a perfected trading system.

## Features

- Manual and automatic **Buy**, **Short**, **Close Buy**, and **Close Short** actions
- Per-stock controls, hotkeys, search, sorting, hide/show, and exclude/allow settings
- Configurable automatic actions, protective exits, cooldowns, position limits, and exposure controls
- Live prices, price-change indicators, chart, order/activity log, and session profit/loss estimate
- Historical supervised training while market updates and trading continue
- Training reports and session logs with `NEW_` filenames, local date/time, and session elapsed time
- Persistent configuration and validated model data
- Screen Stocks skin manager with presets, image import, and skin-pack ZIP import/export
- Setup wizard scans BepInEx plug-in DLL names and flags possible trading-bot conflicts for review
- Optional local Discord quote-feed file for another application to read; the bot itself does not post to Discord
- Optional launch with the game through a small BepInEx startup companion

## How it works

The bot reads the game's released-stock prices and history from its supported `mods/export` files. It submits trades by writing command files under `mods/commands`. It does not connect to the game's server or use game-memory hooks. The game must have its Mods, Export, and Commands features enabled.

The C# dashboard polls the local market export at 250 ms intervals. The game's own price-update cadence controls when new prices become available, so polling faster does not make the game generate prices faster. Training runs in the background and only applies a candidate model if it passes the validation checks; that still does **not** make the strategy reliable or loss-proof.

## Install the ready-to-use package

1. Download the latest release ZIP from the repository's **Releases** page. The current package is also in [`CSharpModBot`](CSharpModBot/) as `ScreenStocksMasterBot_Autostart_CSharp.zip`.
2. Extract the **entire ZIP** to a regular folder.
3. Double-click `ScreenStocksSetup.exe` and approve the Windows administrator prompt. Choose the Screen Stocks Demo folder if it is not at the default Steam path.
4. In the wizard, open the official Screen Stocks mod-support instructions and enable Mods in the game's settings. Check the wizard's acknowledgement box when done.
5. If BepInEx is missing, use the wizard's BepInEx release link to download the official `BepInEx_win_x64_5.4.23.5.zip`. Extract its contents into the game folder beside `Screen Stocks.exe`, return to the wizard, and click **Rescan**. The wizard detects BepInEx before continuing.
6. Review the plug-in DLL list and possible trader warnings. Click **Install bot** when the wizard reports it is ready. It places the bot files, backs up and configures BepInEx logging and autostart, and enables the game's Export, Commands, and Skins mod-file options.
7. Start the game from Steam. The dashboard should open automatically after BepInEx loads. The bot starts in **MANUAL** mode; check that live prices and holdings appear, then test manual controls before enabling AUTO.

The ready-to-use app and wizard require **Windows x64 and the .NET 10 Desktop Runtime**. The automatic game-start companion requires BepInEx 5 in the game folder. Keep all files from the package together; neither `.exe` is a single-file standalone executable. The wizard lets you browse to another game location.

The setup wizard reads DLL filenames and available assembly/product/version metadata under `BepInEx\plugins`; it does not load other mods' code. Matching is heuristic, so it can miss a bot or flag an unrelated stock utility. It never disables or removes another mod. It flags the supplied `ScreenStocksBridge.dll` for review as a bridge that may include trade features. Static strings in that supplied DLL include `TradeService` and `shortTrade`; this is not a full code audit and does not establish exactly when it submits trades. Check that mod's documentation/settings before using it with AUTO. Keep only one active trade-capable system to prevent duplicate orders.

### Manual installation

Close Screen Stocks first. Enable official Mods support using the linked [game instructions](https://store.steampowered.com/news/app/4356780/view/707784625245652851). If BepInEx is not installed, download the official [BepInEx 5.4.23.5 release](https://github.com/BepInEx/BepInEx/releases/tag/v5.4.23.5), choose `BepInEx_win_x64_5.4.23.5.zip`, and extract its contents into the game root (the folder containing `Screen Stocks.exe`). Then copy these items from the complete bot package:

- Copy `ScreenStocksModBotAutostart.dll` (or the identical copy at `BepInEx\plugins\ScreenStocksModBotAutostart.dll` in the package) into `<game folder>\BepInEx\plugins\`.
- Copy the complete package folder `BepInEx\plugins\ScreenStocksModBot` into `<game folder>\BepInEx\plugins\`. Keep every `.exe`, `.dll`, `.json`, and `.ps1` companion file in this folder.
- In Screen Stocks settings, enable Mods. The package installer also enables the `export`, `commands`, and `skins` options in `mods\mod-settings.json`; for manual setup, check these options in the game's mod settings if they are available.

The wizard backs up and configures the BepInEx disk log and the bot's `LaunchBot=true` setting. For a fully manual configuration, set `Enabled = true` and `AppendLog = true` under `[Logging.Disk]` in `BepInEx\config\BepInEx.cfg`, and create `BepInEx\config\com.hetric.screenstocks.modbot.autostart.cfg` containing `[Startup]` and `LaunchBot = true`. These are not needed if you start the dashboard manually every time.

Enable Mods in the game, then launch Screen Stocks. BepInEx opens the dashboard on game startup. To start it yourself, double-click `StartBot.bat`, run `StartBot.ps1`, or double-click `ScreenStocksModBot.exe` from the installed `ScreenStocksModBot` folder. The game and dashboard need to be running for live trading.

To uninstall, run `Uninstall-ScreenStocksModBot.ps1` from the package. It removes the installed program files while preserving the bot's settings, logs, training data, and skins in the game's LocalLow data folder.

## Open the dashboard from PowerShell

After installation, open PowerShell in the installed bot folder:

```powershell
cd "C:\Program Files (x86)\Steam\steamapps\common\Screen Stocks Demo\BepInEx\plugins\ScreenStocksModBot"
.\StartBot.ps1
```

Or run the script directly from a PowerShell window:

```powershell
powershell.exe -ExecutionPolicy Bypass -File "C:\Program Files (x86)\Steam\steamapps\common\Screen Stocks Demo\BepInEx\plugins\ScreenStocksModBot\StartBot.ps1"
```

The launcher avoids opening a second copy if the bot is already running. You can also double-click `ScreenStocksModBot.exe` in that same folder.

## Compatibility with other mods

The setup scan lists DLLs found under `<game folder>\BepInEx\plugins` and labels likely trading/bot-related files for review. For example, old `ScreenStocksMaster.dll` or `Screen Stocks Integrated AI Bot` plugins may compete with this bot if they also submit trades. The scan is a filename/metadata hint, not a guarantee; read the other mod's documentation to confirm what it does. Disable a duplicate trader yourself before enabling AUTO. The installer will not modify other mods. A data/Discord bridge that only reads the bot's local `mods\discord-price-feed.json` should be separate from order execution, but bridge behavior must be checked individually.

The dashboard, UI, trading rules, and training code in this repository are our own. Other people's DLLs are detected and left separate; their code/UI is not copied into this bot. Optional integrations are added only when their behavior and license are understood.

## Build from source

The C# dashboard project is in [`CSharpModBot`](CSharpModBot/). Open PowerShell there and run:

```powershell
.\Build-CSharpBot.ps1
.\Package-CSharpBot.ps1
```

The build uses the installed .NET SDK and creates the dashboard and startup companion package. Normal installation does not require Python, Visual Studio, or a compiler.

## Data locations

The default game mod-data directory is:

```text
%USERPROFILE%\AppData\LocalLow\Conradical Games\Screen Stocks\mods
```

- Market snapshots and price history: `mods\export\`
- Trade commands: `mods\commands\<stock ID>\`
- Custom skins: `mods\skins\`
- Settings: `mods\screen_stocks_bot.json`
- Bot session logs and training reports: the game's existing `Logs` folder; bot-generated files begin with `NEW_`
- Quote feed for an optional Discord-side bridge: `mods\discord-price-feed.json`

## Important notes

- Disable other Screen Stocks trading bots before using this one. Running multiple traders can cause conflicting orders.
- AUTO allocation defaults to 100%, but available cash, position limits, exposure caps, cooldowns, and other configured gates affect actual orders. Review those settings before enabling AUTO.
- Emergency Stop pauses new automatic orders; it does not automatically close existing positions.
- The dashboard's profit/loss is an estimate based on positions tracked by the bot during the current session. It is not a guarantee or a complete account statement.
- The Discord quote file is only a local data feed. A separate Discord integration must read it and edit a message; this project does not make Discord network requests. Inspect bridge DLLs before using their trade controls alongside this bot.

## Project status

This is an actively developed community project. The bot, trainer, game-file integration, and UI may change between versions. Please report issues with the game version, bot package version, relevant `NEW_` logs, and the exact steps to reproduce them. Remove private identifiers or any webhook secrets before sharing logs.
