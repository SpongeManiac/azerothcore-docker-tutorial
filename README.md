# Azerothcore Docker Install
## Prerequisites & Dependencies

Before starting on this guide, be sure to have the following hardware and software installed:

*   **SSD with > 100GB capacity for DB**
*   Docker Desktop _**OR**_ Docker & Docker Compose
    *   **For Windows:**
        *   Install [Docker Desktop](https://www.docker.com/products/docker-desktop/), which bundles Docker and Docker Compose into a desktop app
    *   **For Linux:**
        *   Install Docker using the [Official Docker Install Guide](https://docs.docker.com/engine/install/), which should include Docker Compose.
*   Git
    *   **For Windows:**
        *   Install [Git](https://git-scm.com/install/), which is a tool for version control and downloading source code.
    *   **For Linux:**
        *   Install Git using your distro's package manager.

> [!CAUTION]
> **When installing Git on Windows, be sure to associate** `**.sh**` **files with bash. This will allow you to execute the scripts by double-clicking them.** If you don't, you will have to either execute commands manually or re-install Git with this option enabled.

> [!NOTE]
> Some distros keep Docker and Docker Compose in separate packages, so be sure `docker compose` is a recognized command. You may also need to install the [buildx docker plugin](https://github.com/docker/buildx?tab=readme-ov-file#linux-packages) if `docker compose build` fails.

## Setup & Configure

> [!IMPORTANT]
> If you want to run this stack using **Portainer**, see the **Portainer Setup**.

Create a clean directory in a place of your choosing on an SSD of sufficient size **(100GB+)**. I will be using `…/wow/` for this guide. We will call this the **WoW Root**. This will be the root of all files and folders for the Azerothcore server. Here is an overview of how your project will look in the end:

```plain
wow/ <- WoW Root
|-- ac-client-data/ <- Client Data Directory
|   \-- ...
|
|-- ac-database/ <- Database Directory
|   \-- ...
|
\-- playerbots-calvincore/ <- Repository Root
    |-- modules/ <- Modules Directory
    |-- docker-compose.yaml
    |-- .env
    \-- ...
```

> [!TIP]
> _**IF**_  **you enabled associating shell scripts with bash,** this is the only command to run in the terminal. You can close it once it is finished. Later in the guide, you can double-click the `.sh` files to run them.  
>   
> **Otherwise**, you will need to re-install git, or if you know what you are doing, just open Git Bash and execute the `.sh` files manually. Be sure to execute the scripts from the directory they reside in.

1.  Navigate to the **Wow Root** and create a terminal or PowerShell in that directory. Then execute the following command to download the server source:
    
    ```plain
    git clone https://github.com/SpongeManiac/playerbots-calvincore
    ```
2.  Navigate into the **Repository Root** (The folder created from the clone command) and create a copy of `.env.example` and rename it to `.env`. If the file is not visible, be sure to enable hidden files for your system.
3.  Modify the `.env` file and fill in your own config values and credentials. Be sure to update your DB Root password to something secure.
    
    Here are the contents of `.env.example`:
    
    ```plain
    DOCKER_DB_EXTERNAL_PORT=3306
    DOCKER_DB_ROOT_PASSWORD=32_length_no_special_chars_pass_
    DOCKER_VOL_DB=../ac-database
    DOCKER_IMAGE_TAG=local
    DOCKER_USER_ID=1000
    DOCKER_GROUP_ID=1000
    DOCKER_USER=acore
    DOCKER_VOL_ETC=./env/dist/etc
    DOCKER_VOL_LOGS=./env/dist/logs
    DOCKER_AC_ENV_FILE=conf/dist/env.ac
    DOCKER_WORLD_EXTERNAL_PORT=8085
    DOCKER_SOAP_EXTERNAL_PORT=7878
    DOCKER_VOL_DATA=../ac-client-data
    DOCKER_AUTH_EXTERNAL_PORT=3724
    DOCKER_AC_CLIENT_FOLDER=./var/client
    DOCKER_VOL_ROOT=.
    ```
4.  Navigate to the `/wow/playerbots-calvincore/modules/` folder and modify the `modules.txt` file. Add the git repository URLs on each line for the modules you want to play with.
    
    For example, these are the default modules for Calvincore:
    
    ```sh
    https://github.com/azerothcore/mod-account-achievements.git:bfbe3677635feeef823057964e028e023633115a
    https://github.com/azerothcore/mod-ah-bot.git:a680cc1c98290713e9b3d3289544af78e5186dc1
    https://github.com/azerothcore/mod-ale.git:1ab53bac10a784e9a18e15d2be3b6d0603c91580
    https://github.com/abracadaniel22/mod-fly-anywhere.git:d8c8ea76bb7ef6aea395527ec42aff43c6e02799
    https://github.com/azerothcore/mod-guildhouse.git:f7e225a3401bc363b0ece5f513eca1d9785236b7
    https://github.com/azerothcore/mod-low-level-rbg.git:fd6077de0fd49bf2caaae3c5c4dcb857178cf7b9
    https://github.com/DustinHendrickson/mod-ollama-chat.git:186d6a120a025729b430c7c42e7c5d3ad8aa7833
    https://github.com/DustinHendrickson/mod-player-bot-guildhouse.git:1d4d649982a523b674321f6ca4c23d11e60c1b7b
    https://github.com/mod-playerbots/mod-playerbots.git:c8dce882d621f6250a96370bfd3c1bf79e9ebc4e
    https://github.com/azerothcore/mod-solo-lfg.git:3821fe1d108ade8d2b7ad6611e41154e05864c65
    ```

> [!CAUTION]
> **Be absolutely sure that both** `DOCKER_VOL_DB` **and** `DOCKER_VOL_DATA` **point to a path on an SSD** **with a sufficient amount of storage space (100GB+).** The config values `DOCKER_VOL_DB` and `DOCKER_VOL_DATA` are used to define where the database and client data folders/volumes will be created.
> 
> **DO** _**NOT**_ point the volumes to any sub-folder of the repository because it is used as a volume. The DB and client data folders must be in a separate location. The default is to place these one level above the repository folder:
> 
> ```plain
> wow/ <- WoW Root
> |-- ac-client-data/ <- Client Data Directory
> |   \-- ...
> |
> |-- ac-database/ <- Database Directory
> |   \-- ...
> |
> \-- playerbots-calvincore/ <- Repository Root
>     |-- modules/ <- Modules Directory
>     |-- docker-compose.yaml
>     |-- .env
>     \-- …
> ```

> [!NOTE]
> You can specify a particular commit for a repository with the format `[repo_url]:[commit_hash]`:
> 
> ```sh
> https://github.com/azerothcore/mod-ale.git:ecb3d102ac056c8ee92c3778158b475e16650ce2
> ```

## Building Images

Before executing these commands, be sure you have configured the repository to your liking using the previous section.

You can now build the docker images by executing the `/wow/playerbots-calvincore/docker-build.sh` file.

> [!CAUTION]
> **Watch for errors in the output of this command.** Docker likes to hide the command output to keep things tidy, but this can be annoying when debugging. To view the full output, check the build log: `/wow/playerbots-calvincore/build.log`
> 
> If you have errors, check to see if any updates have been made recently to a module or the root repository. If a module is broken due to changes, you can use a specific commit hash in the `modules.txt` file.

## Populate DB

The database must be populated with game data and migrations in order to function properly. In addition to this, modules often have custom SQL files that need to be applied to a database before running the world server. This section will go over populating the DB and applying module SQL files.

> [!CAUTION]
> **Watch for errors in the DB and DB import.** If you have made any mistakes or errors in your `.env` config, they will manifest at this step. One common issue is having special characters in your DB password. Avoid problematic symbols such as ``!@#$%^&()[]{}|*=+<>/\?`~:;'"``. Dashes, underscores, periods, and commas should be fine. 
> 
> If you need to change the DB password, login to the DB using Adminer and follow any guide for modifying the root password in MySQL. If you cannot log into the DB from Adminer, you may need to log into the MySQL DB from inside the container:
> 
> ```plain
> # Make sure container is running
> docker compose up -d ac-client-data-init ac-database ac-db-import adminer
> # Exec into the running container
> docker exec -it ac-database mysql -u root -p
> 
> # Then run inside MySQL:
> ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';
> FLUSH PRIVILEGES;
> ```

> [!IMPORTANT]
> The database will not fully populate until it is run at least once. Each time you build the server, be sure to execute `docker_run.sh` at least once before executing module SQL.

**From the Repository Root (**`/wow/playerbots-calvincore/`**):**

1.  Be sure to let the server completely boot at least once and execute `docker_run.sh`.
2.  Once the server has completely booted and all SQL has been applied, stop the server with `ctrl+c`.

## Connecting to Adminer

> [!TIP]
> If you are running the docker containers on another host, you can connect to Adminer using `IP:Port`:
> 
> ```http
> http://192.168.1.18:8080/
> ```

1.  To stage changes to the SQL server, execute the `db-staging.sh` file. This will only bring up the db and db import containers so that you can execute module SQL.
    
    Wait for the DB Import process to complete. It will be complete when you see the following line in the console log:
    
    ```plain
    ac-adminer   | [Sat Mar 28 17:24:50 2026] PHP 8.4.19 Development Server (http://[::]:8080) started
    ```
2.  Open the Adminer web page in a browser. You will need to enter the `IP:Port` of the server running the docker containers. In my case, I'm running them on my desktop so I can use `localhost` instead of an IP address:
    
    ```http
    http://localhost:8080
    ```
3.  Enter the credentials for your `root` DB user. Use the password that was set in the `.env` file:
    
    <table><tbody><tr><td><ul style="list-style-type:disc;"><li data-list-item-id="e7384211dbbf893d193ae27b45f066a02">Ensure <code>System</code> dropdown is set to <code>MySQL /  MariaDB</code></li><li data-list-item-id="e6fedc4c83b526f4cf62e1522843fe23e">Ensure <code>Server</code> is set to the name of your DB container, <code>ac-database</code> by default</li><li data-list-item-id="e9bc228e79a278fd74792d7b7ea0aa668">Ensure <code>Username</code> is set to <code>root</code></li><li data-list-item-id="e4b5afc9732e33bfa4cb5c1fb93ac3b61">Paste the password from your <code>.env</code> file</li><li data-list-item-id="e2754a0b575b4123b463798f6335281ec">Ensure <code>Database</code> is empty</li><li data-list-item-id="e6bbe469c5cfe8fc9a785df5ab3c0b8d1">(Optional) Check <code>Permanent login</code></li><li data-list-item-id="e15e5cc5be393dc4130d7c33ac30b358d">Click Login</li></ul></td><td><p>&nbsp;</p><figure class="image image-style-align-right"><img style="aspect-ratio:452/443;" src="3_Azerothcore Docker Install.png" width="452" height="443"></figure></td></tr></tbody></table>
4.  Once successfully logged in, you should see the following screen:
    
    <figure class="image"><img style="aspect-ratio:615/413;" src="Azerothcore Docker Install.png" width="615" height="413"></figure>

## Apply Module SQL

> [!CAUTION]
> **Some modules do not follow the typical setup and require special instructions to properly install.** Review the module's Github page and _**carefully**_ go over the install instructions.
> 
> **Some modules do not follow the expected folder structure for SQL files.** Thoroughly inspect each folder to be sure a module does not contain any SQL to be executed.

1.  Navigate to the `/modules/` folder and pick a module to start with.
2.  Inspect the folder structure of the module. If the module has SQL to apply, it will have the path `/data/sql` like in this example using the Guild House Module:
    
    ```plain
    playerbots-calvincore/ <- Repository Root
    \-- modules/ <- Modules Folder
        \-- mod-guildhouse/ <- Guildhouse Module
            \-- data/
                \-- sql/
                    |-- db-world/ <- These SQL files are for the world DB
                    |   |-- 2024_04_07_02_innkeeper.sql
                    |   |-- 2024_04_07_01_guildhouse_spawns.sql
                    |   \-- 2024_04_07_00_creatures_objects.sql
                    |
                    \-- db-characters/ <- These SQL files are for the characters DB
                        \-- 2024_04_07_guildhouse.sql
    ```
    
    If your module does not have a `/data/sql` folder, it most likely means that the module does not require any SQL to be executed and you can skip it.
3.  The folder will tell you which database the SQL files should be executed in. You must execute each SQL file in chronological order, starting with the oldest files first.
    1.  Switch back to the Adminer web page. Click the `DB` dropdown and select the database to execute SQL in:
        
        <figure class="image"><img style="aspect-ratio:610/422;" src="4_Azerothcore Docker Install.png" width="610" height="422"></figure>
    2.  Once the proper database is selected, click the `Import` link:
        
        <figure class="image"><img style="aspect-ratio:610/422;" src="6_Azerothcore Docker Install.png" width="610" height="422"></figure>
    3.  Now click the `Browse…` button and select the SQL files to be executed:
        
        <figure class="image"><img style="aspect-ratio:960/734;" src="2_Azerothcore Docker Install.png" width="960" height="734"></figure>
    4.  Finally, click `Execute` button:
        
        <figure class="image"><img style="aspect-ratio:560/403;" src="7_Azerothcore Docker Install.png" width="560" height="403"></figure><figure class="image"><img style="aspect-ratio:560/403;" src="1_Azerothcore Docker Install.png" width="560" height="403"></figure>
4.  Repeat steps **6**, **7**, and **8** for each database in each module until all modules are done. Once finished, use `ctrl+c` to stop the containers.

> [!IMPORTANT]
> If you get any errors, you may need to check the module and see if it was updated recently. Incompatibilities in the SQL structure will cause errors.

## Configure Server & Modules

Now that the containers are built and the database is ready, we must configure the server settings and module settings.

1.  Ensure each `.dist` file has been duplicated and renamed to be `.conf` under the following paths:
    
    *   `/playerbots-calvincore/env/dist/etc/`
    *   `/playerbots-calvincore/env/dist/etc/module/`
    
    <table><tbody><tr><td>Server Configs</td><td>&nbsp;</td><td>&nbsp;</td></tr><tr><td><pre><code class="language-text-plain">playerbots-calvincore/env/dist/etc/
    |-- worldserver.conf.dist
    |-- dbimport.conf.dist
    \-- authserver.conf.dist</code></pre></td><td>→</td><td><pre><code class="language-text-plain">playerbots-calvincore/env/dist/etc/
    |-- worldserver.conf.dist
    |-- worldserver.conf
    |-- dbimport.conf.dist
    |-- dbimport.conf
    |-- authserver.conf.dist
    \-- authserver.conf</code></pre></td></tr><tr><td>Server Modules</td><td>&nbsp;</td><td>&nbsp;</td></tr><tr><td><pre><code class="language-text-plain">playerbots-calvincore/env/dist/etc/modules
    |-- fly-anywhere.conf.dist
    |-- low_level_rbg.conf.dist
    |-- mod_achievements.conf.dist
    |-- mod_ahbot.conf.dist
    |-- mod_ale.conf.dist
    |-- mod_guildhouse.conf.dist
    |-- mod_ollama_chat.conf.dist
    |-- mod_player_bot_guildhouse.conf.dist
    |-- playerbots.conf.dist
    \-- SoloLfg.conf.dist</code></pre></td><td>→</td><td><pre><code class="language-text-plain">playerbots-calvincore/env/dist/etc/modules
    |-- fly-anywhere.conf
    |-- fly-anywhere.conf.dist
    |-- low_level_rbg.conf
    |-- low_level_rbg.conf.dist
    |-- mod_achievements.conf
    |-- mod_achievements.conf.dist
    |-- mod_ahbot.conf
    |-- mod_ahbot.conf.dist
    |-- mod_ale.conf
    |-- mod_ale.conf.dist
    |-- mod_guildhouse.conf
    |-- mod_guildhouse.conf.dist
    |-- mod_ollama_chat.conf
    |-- mod_ollama_chat.conf.dist
    |-- mod_player_bot_guildhouse.conf
    |-- mod_player_bot_guildhouse.conf.dist
    |-- playerbots.conf
    |-- playerbots.conf.dist
    |-- SoloLfg.conf
    \-- SoloLfg.conf.dist</code></pre></td></tr></tbody></table>
2.  Go through each `.conf` file in both directories and apply settings to your liking.

> [!IMPORTANT]
> **Some modules will only work if enabled through the config.** Additionally, modules such as the Auction House may require extra steps such as setting up a dedicated auction house character.

## Run Server

The server is finally ready to be started. You can either run the server in the background, or you can run the server in the console to observe the output. I recommend running the server in the console at least once to ensure no errors occur during startup.

**From the Repository Root (**`/wow/playerbots-calvincore/`**):**

*   Start in console:
    *   Windows:
        *   Run `console_start.bat`
    *   Linux:
        
        ```plain
        docker compose up
        ```
*   Start in background:
    *   Windows:
        *   Run `background_start.bat`
    *   Linux:
        
        ```plain
        docker compose up -d
        ```

> [!WARNING]
> When running the server in the console, **do not press** `**ctrl+c**` **to detach**, as this will stop all of the containers and servers. Instead, to detach from the server output _**without**_ stopping the server, use the key combination `ctrl+a` then `ctrl+d`. **Order matters!**

## Game Server

Before you can log into the server, you must add an account for yourself and anyone else who wants to play. You can do this by attaching to the Game Server.

**From the Repository Root (**`/wow/playerbots-calvincore/`**):**

*   Execute the following command:
    
    ```plain
    docker attach ac-worldserver
    ```
    
    *   To add an account, run the following command:
        
        ```plain
        account create <username> <password>
        ```
        
        Example:
        
        ```plain
        account create sillygoose 900$3
        ```

> [!CAUTION]
> **Do** _**NOT**_ **use** `**ctrl+c**` **to detach from the world server!** When finished executing commands on the game server, use the key-combo `ctrl+p` then `ctrl+q` to safely detach without closing the server.
