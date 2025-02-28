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
| `user_game_ids` | array | List of Game IDs the account is associated with. (Usually only one) |
| `login_types` | array | List of Login Types the account uses (Usually only one) |
| `access_token` | string | Encrypted Access Token the Client uses for several cases. Contains an Encrypted JSON. More below.
| `token_type` | string | Token Type used (Always `bearer`). |
| `expires_at` | long | When it will expire (Current Unix Time + 1200) |

Decrypted Access Token:
| Name | Type | Description |
| --- | --- | --- |
| `account_id` | string | Account ID associated with you |
| `user_game_ids` | array | List of Game IDs the account is associated with. Usually, you only have one profile, so you only have one ID. |
| `g` | int | Game ID associated with the platform you're playing on (1 or 27) |
| `token_version` | int | Version the Token is using (always 1) |
| `generated_on` | long | When the Token was generated (Current Unix Time) |
| `expires_at` | long | When it will expire (Current Unix Time + 1200) |
| `username` | string | Account Login Username |
| `login_type` | string | Current Login Type |

>[!NOTE]
>`user_game_id`, `username` and `account_id` are **completely seperate** do **not** mistake them for eachother. 

3. Afterwards, MSM sends a POST request to `msmpc.bbbgame.net/pregame_setup` (`msm-auth.bbbgame.net/pregame_setup` for mobile), and if it's successful, it'll respond with several elements:

| Name | Type | Description |
| --- | --- | --- |
| `ok` | boolean | `true` if successful, `false` if not. All other params will not be sent if `false` |
| `serverId` | int | Specific ID for the AWS instance that uses `serverIp` |
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
| `last_updated` | long | (Not Required) Client Last Update |
| `last_update_version` | string | Client Last Updated Version |
| `client_device` | string | (Not Required) Client Device ID |
| `client_os` | string | (Not Required) Client Device OS Version |
| `client_platform` | string | Client MSM Platform (Mobile or PC) |
| `client_subplatform` | string | (Not Required) Client MSM Sub-Platform (Steam) |
| `raw_device_id` | string | (Not Required) Raw Client Device ID |
| `client_lang` | string | Language the Client prefers |

7. If all goes well, the server should add multiple elements to your User and Session Object and send `gs_initialized` with one parameter, your `bbb_id`.

New User and Session Params:
| Name | Type | Description |
| --- | --- | --- |

9. Now MSM needs to download all the [Static Data](https://www.indeed.com/career-advice/career-development/static-data-vs-dynamic-data). MSM sends 1 param for all of them, `last_updated`, although it is not required. You can view all Requests MSM sends [here](https://github.com/riotschoolacc/MSM-Server-Tools/blob/main/requests.md).

10. After downloading all the Static Data and updating it in the Cache in your User `%AppData%/LocalLow/`, they must finish up. MSM will send `gs_player` to get all Player Data, then process any Previous Purchases, and finally load your Player Data.
