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

4. Next, MSM will attempt to download the new files via the `contentUrl` and store them in your User `%AppData%/LocalLow/`.

>[!NOTE]
>Reminder that MSM uses Smartfox2X for all it's Client➔Server necessities.

5. Now, MSM has to connect to the Game Servers. It'll attempt to connect at `(serverIp):9933`.

6. If it's able to connect, MSM will attempt to Login using Params:

| Name | Type | Description |
| --- | --- | --- |
| `username` | string | `user_game_id` from the Auth Token response |
| `password` | string | Your password. Depends on your `login_type` |
| `zone` | string | The Smartfox Zone to join |
| `auth_info` | SFSObject | Login Params |

Login Params:
| Name | Type | Description |
| --- | --- | --- |
| `token` | string | `access_token` from the Auth Token response |
| `access_key` | string | A hardcoded string inside the MSM EXE |
| `client_version` | string | The current Client Version |

7. If all goes well, the server should add multiple elements to your User Object and send `gs_initialized` with one parameter: your `bbb_id`.

8. Now MSM needs to download all the [Static Data](https://www.indeed.com/career-advice/career-development/static-data-vs-dynamic-data). MSM sends 1 param for all of them, `last_updated`, although it is not required. You can view all Requests MSM sends [here](https://github.com/riotschoolacc/MSM-Server-Tools/blob/main/requests.md).

9. After downloading all the Static Data and updating it in the Cache, they must finish up. MSM will send `gs_player` to get all Player Data, then process any Previous Purchases, and finally load your Player Data.
