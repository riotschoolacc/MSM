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

1. The first thing MSM does when you click play is attempt to log in to Steam **only if** your `login_type` is `steam`. 

2. The next thing MSM does when you click play is an attempt to authorize your account by sending a POST request to `auth.bbbgame.net/auth/api/token/`. If it's successful, the server will respond with several elements:

| Name | Type | Description |
| --- | --- | --- |
| `ok` | boolean | `true` if successful, `false` if not. All other params will not be sent if `false` |
| `user_game_ids` | array | List of Game IDs the account is associated with. (Usually only one) |
| `login_types` | array | List of Login Types the account uses (Usually only one) |
| `access_token` | string | Encrypted Access Token the Client uses for several cases. Contains an Encrypted JSON. More below.
| `token_type` | string | Token Type used (Always `bearer`). |
| `expires_at` | long | When it will expire (Current Unix Time + 1200, or 20 Minutes.) |

Decrypted Access Token:
| Name | Type | Description |
| --- | --- | --- |
| `account_id` | string | Account ID associated with you |
| `user_game_ids` | array | List of Game IDs the account is associated with. Usually, you only have one profile, so you only have one ID. |
| `g` | int | Game ID associated with the platform you're playing on (1 or 27) |
| `token_version` | int | Version the Token is using (always 1) |
| `generated_on` | long | When the Token was generated (Current Unix Time) |
| `expires_at` | long | When it will expire (Current Unix Time + 1200, or 20 Minutes.) |
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
| `token` | string | (Required) `access_token` from the Auth Token response |
| `access_key` | string | (Required) A hardcoded string inside the MSM EXE |
| `client_version` | string | (Required) The current Client Version |
| `last_updated` | long | (Not Required) Client Last Update Unix Timestamp |
| `last_update_version` | string | (Required) Client Last Updated Version |
| `client_device` | string | (Not Required) Client Device ID |
| `client_os` | string | (Not Required) Client Device OS Version |
| `client_platform` | string | (Required) Client MSM Platform (Mobile or PC) |
| `client_subplatform` | string | (Not Required) Client MSM Sub-Platform (Steam) |
| `raw_device_id` | string | (Not Required) Raw Client Device ID |
| `client_lang` | string | (Required) Language the Client prefers |

#### Even if a param is not required, the Client still sends it. 

7. If all goes well, the server should add multiple elements to your Session Object via the Decrypted Token.

New Session Params:
| Name | Type | Description |
| --- | --- | --- |
| `user_game_id` | string | Your `user_game_id` extracted from your Token |
| `username` | string | Your `username` extracted from your Token |
| `loginType` | string | Your current `login_type` extracted from your Token |
| `access_key` | string | The `access_key` from your Login Params |
| `client_version` | string | The `client_version` from your Login Params |
| `last_updated` | long | The `last_update` from your Login Params |
| `last_update_version` | string | The `last_update_version` from your Login Params |
| `client_device` | string | The `client_device` from your Login Params |
| `client_platform` | string | The `client_platform` from your Login Params |
| `client_subplatform` | string | The `client_subplatform` from your Login Params |
| `client_os` | string | The `client_os` from your Login Params |
| `raw_device_id` | string | The `raw_device_id` from your Login Params |
| `client_lang` | string | the `client_lang` from your Login Params |

#### If some Params aren't in your Login Data, it will set the Session Param for it to an empty string.

8. Now, MSM attempts to login to the `zone`. The server will then extract some of the params from the Session Object and add them to the User Object, and some other params not from the Session. *This is because User Objects aren't created until after the Login process, but Sessions are.*

New User Params:
| Name | Type | Description |
| --- | --- | --- |
| `player_object` | SFSObject | Player Data SFSObject |
| `bbb_id` | long | Users BBB ID |
| `client_version` | string | The `client_version` from your Session Params |
| `last_update_version` | string | The `last_update_version` from your Session Params |
| `client_device` | string | The `client_device` from your Session Params |
| `client_platform` | string | The `client_platform` from your Session Params |
| `client_subplatform` | string | The `client_subplatform` from your Session Params |
| `client_os` | string | The `client_os` from your Session Params |
| `ip_address` | string | The Users IP Address |
| `client_lang` | string | the `client_lang` from your Session Params |
| `idfv` | string | (**Only if `client_platform` is `ios`**) The `raw_device_id` from your Session Params |
| `android_id` | string | (**Only if `client_platform` is `android`**) The `raw_device_id` from your Session Params |

9. After the User has joined the `zone`, the Server will send `gs_initialized` with one param, your `bbb_id`.

10. Then, the Server will send all `user_game_settings`.

11. Now MSM needs to download all the [Static Data](https://www.indeed.com/career-advice/career-development/static-data-vs-dynamic-data). MSM sends 1 param for all of them, `last_updated`, although it is not required. You can view all Requests MSM sends [here](https://github.com/riotschoolacc/MSM-Server-Tools/blob/main/requests.md).

12. After downloading all the Static Data and updating it in the Cache in your User `%AppData%/LocalLow/`, they must finish up. MSM will send `gs_player` to get all Player Data, then process any Previous Purchases, and finally load your Player Data.

If you were able to Stomach all of that, good for you, but now it's time for the Real Game.
