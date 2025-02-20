# MSM
Repo that teaches you what's going on internally in My Singing Monsters.

Preload
-
Before you do anything when you open MSM, the game has to:

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
| user_game_ids | table | List of Game IDs the account is associated with. Usually, you only have one profile, so you only have one ID.
| 
