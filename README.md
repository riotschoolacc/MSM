# MSM
Repo that teaches you what's going on internally in My Singing Monsters.

Preload
-
Before you do anything, when you open MSM, the game has to:

* Load all Loading Screen PNGs/AVIFs

* Load all Loading Screen XML Files

* Load all Loading Screen LUA Files

Then, we can continue.

Play
-
When you click play, MSM does several things.

1. The first thing MSM does when you click play is attempt to log in to Steam. If the game says "Steam Authorization failed, " close MSM and try to launch it via Steam.

2. The next thing MSM does when you click play is an attempt to authorize your account by sending a POST request to `auth.bbbgame.net/auth/api/token/`. If it's successful, the server will respond with several elements:

| Name | Type | Description |
| --- | --- | --- |
| `ok` | boolean | `true` if successful, `false` if not. All other params will not be sent if `false` |
| `user_game_ids` | table | List of Game IDs the account is associated with. Usually, you only have one profile, so you only have one ID. |
| `login_types` | table | List of Login Types the account uses |
| `access_token` | string | Encrypted Access Token the Client uses for several cases. Contains an Encrypted JSON with your Account ID `account_id`, User Game IDs (same as the one in the response), Game ID `game` (1 or 27), Token Version `token_version` (1), Generated On `generated_on` (Current Unix Time) Expires At `expires_at`, Username `username` and Login Type `login_type`
| `token_type` | string | Token Type used (Always `bearer`). |
| `expires_at` | long | Current Unix Time + 1200 |

3. Afterwards, MSM sends a POST request to `msmpc.bbbgame.net/pregame_setup` (`msm-auth.bbbgame.net/pregame_setup` for mobile), and if it's successful, it'll respond with several elements:

| Name | Type | Description |
| --- | --- | --- |
| `ok` | boolean | `true` if successful, `false` if not. All other params will not be sent if `false` |
| `serverId` | int | Specific ID for the `serverIp` |
| `serverIp` | string | IP that the Client will use to connect to |
| `contentUrl` | string | URL that MSM downloads update content from |
