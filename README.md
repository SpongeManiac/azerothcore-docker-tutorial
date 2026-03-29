# Docker Install
## Custom Azerothcore W/ Modules

### Prerequisites & Dependencies

Before starting on this guide, be sure to have the following hardware and software installed:

*   **SSD with > 100GB capacity for DB**
*   Docker Desktop _**OR**_ Docker & Docker Compose
    *   **For Windows:**
        *   Install [Docker Desktop](https://www.docker.com/products/docker-desktop/), which bundles Docker and Docker Compose into a desktop app
    *   **For Linux:**
        *   Install Docker using the [Official Docker Install Guide](https://docs.docker.com/engine/install/), which should include Docker Compose.
        *   > [!NOTE]
            > Some distros keep Docker and Docker Compose in separate packages, so be sure `docker compose` is a recognized command. You may also need to install the [buildx docker plugin](https://github.com/docker/buildx?tab=readme-ov-file#linux-packages) if `docker compose build` fails.
*   Github Desktop **OR** Git
    *   **For Windows:**
        *   Install [Github Desktop](https://desktop.github.com/download/) which includes Git and provides a user interface for the tool**.**
    *   **For Linux:**
        *   Install Git using your distro's package manager.

### Setup & Configure

> [!IMPORTANT]
> If you want to run this stack using **Portainer**, see the **Portainer Setup**.

1.  Start by navigating to a clean directory (I'm using `…/wow/`) and cloning a quick-start Azerothcore fork and entering the repository:
    *   Windows:
        1.  Open Github Desktop
    *   Linux:
        
        ```sh
        git clone https://github.com/SpongeManiac/playerbots-calvincore && cd playerbots-calvincore
        ```
2.  Create `.env` file to store private credentials:
    *   Windows:
        *   Run `make-env.bat`
    *   Linux: 
        
        *   Run `make-env.sh`
3.  Modify the created `.env` file and fill in your own config values and credentials:
    
    > [!CAUTION]
    > The config values `DOCKER_VOL_DB` and `DOCKER_VOL_DATA` are used to define where the database and client data folders/volumes will be created.  
    >   
    > **Be absolutely sure that both** `**DOCKER_VOL_DB**` **and** `**DOCKER_VOL_DATA**` **point to a path on an SSD** **with a sufficient amount of storage space (100GB+).**
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
    >     \-- ...
    > ```
    
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
    https://github.com/azerothcore/mod-account-achievements.git
    https://github.com/azerothcore/mod-ah-bot.git
    https://github.com/azerothcore/mod-ale.git
    https://github.com/abracadaniel22/mod-fly-anywhere.git
    https://github.com/azerothcore/mod-guildhouse.git
    https://github.com/azerothcore/mod-low-level-rbg.git
    https://github.com/DustinHendrickson/mod-ollama-chat.git
    https://github.com/DustinHendrickson/mod-player-bot-guildhouse.git
    https://github.com/mod-playerbots/mod-playerbots.git
    https://github.com/azerothcore/mod-solo-lfg.git
    ```
    
    > [!NOTE]
    > You can specify a particular commit for a repository with the format `[repo_url]:[commit_hash]`:
    > 
    > ```sh
    > https://github.com/azerothcore/mod-ale.git:ecb3d102ac056c8ee92c3778158b475e16650ce2
    > ```

### Building Images

Before executing these commands, be sure you have configured the repository to your liking using the previous section.

**From the Repository Root (**`/wow/playerbots-calvincore/`**):**

1.  Build the docker images:
    
    *   Windows:
        *   Run `docker-build.bat`
    *   Linux:
        
        ```plain
        sh docker-build.sh
        ```
    
    > [!CAUTION]
    > **Watch for errors in the output of this command.** Docker likes to hide the command output to keep things tidy, but this can be annoying when debugging. To view the full output, check the build log: `/wow/playerbots-calvincore/build.log`
    > 
    > If you have errors, check to see if any updates have been made recently to a module or the root repository. If a module is broken due to changes, you can use a specific commit hash in the `modules.txt` file.

### Populate DB & Apply Module SQL

The database must be populated with game data and migrations in order to function properly. In addition to this, modules often have custom SQL files that need to be applied to a database before running the world server. This section will go over populating the DB and applying module SQL files.

**From the Repository Root (**`/wow/playerbots-calvincore/`**):**

1.  Run the following docker command to populate and bring up the SQL dashboard:
    
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
    
    *   Windows:
        *   Run `db-staging.bat`
    *   Linux:
        
        ```sh
        sh db-staging.sh
        ```
2.  Wait for the DB Import process to complete. It will be complete when you see the following line in the console log:
    
    ```plain
    ac-adminer   | [Sat Mar 28 17:24:50 2026] PHP 8.4.19 Development Server (http://[::]:8080) started
    ```
3.  Open the Adminer web page in a browser. You will need to enter the `IP:Port` of the server running the docker containers:
    
    ```http
    http://192.168.1.9:8080
    ```
    
    > [!NOTE]
    > If you are running the docker containers on your desktop, you can connect using `localhost`:
    > 
    > ```http
    > http://localhost:8080/
    > ```
4.  Enter the credentials for your `root` DB user. Use the password that was set in the `.env` file:
    
    <table><tbody><tr><td><ul style="list-style-type:disc;"><li data-list-item-id="e7384211dbbf893d193ae27b45f066a02">Ensure <code>System</code> dropdown is set to <code>MySQL /  MariaDB</code></li><li data-list-item-id="e6fedc4c83b526f4cf62e1522843fe23e">Ensure <code>Server</code> is set to the name of your DB container, <code>ac-database</code> by default</li><li data-list-item-id="e9bc228e79a278fd74792d7b7ea0aa668">Ensure <code>Username</code> is set to <code>root</code></li><li data-list-item-id="e4b5afc9732e33bfa4cb5c1fb93ac3b61">Paste the password from your <code>.env</code> file</li><li data-list-item-id="e2754a0b575b4123b463798f6335281ec">Ensure <code>Database</code> is empty</li><li data-list-item-id="e6bbe469c5cfe8fc9a785df5ab3c0b8d1">(Optional) Check <code>Permanent login</code></li><li data-list-item-id="e15e5cc5be393dc4130d7c33ac30b358d">Click Login</li></ul></td><td><p>&nbsp;</p><figure class="image image-style-align-right"><img style="aspect-ratio:452/443;" src="3_Docker Install_image.png" width="452" height="443"></figure></td></tr></tbody></table>
5.  Once successfully logged in, you should see the following screen:
    
    <figure class="image"><img style="aspect-ratio:615/413;" src="Docker Install_image.png" width="615" height="413"></figure>
6.  Now navigate to the `/modules/` folder and pick a module to start with.
    
    > [!CAUTION]
    > **Some modules do not follow the typical setup and require special instructions to properly install.** Review the module's Github page and _**carefully**_ go over the install instructions.
7.  Inspect the folder structure of the module. If the module has SQL to apply, it will have the path `/data/sql` like in this example using the Guild House Module:
    
    > [!CAUTION]
    > **Some modules do not follow the expected folder structure for SQL files, such as the Azerothcore Lua Engine (ALE).** Thoroughly inspect each folder to be sure a module does not contain any SQL to be executed.
    
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
8.  The folder will tell you which database the SQL files should be executed in. You must execute each SQL file in chronological order, starting with the oldest files first.
    1.  Switch back to the Adminer web page. Click the `DB` dropdown and select the database to execute SQL in:
        
        <figure class="image"><img style="aspect-ratio:610/422;" src="4_Docker Install_image.png" width="610" height="422"></figure>
    2.  Once the proper database is selected, click the `Import` link:
        
        <figure class="image"><img style="aspect-ratio:610/422;" src="6_Docker Install_image.png" width="610" height="422"></figure>
    3.  Now click the `Browse…` button and select the SQL files to be executed:
        
        <figure class="image"><img style="aspect-ratio:960/734;" src="2_Docker Install_image.png" width="960" height="734"></figure>
    4.  Finally, click `Execute` button:
        
        <figure class="image"><img style="aspect-ratio:560/403;" src="7_Docker Install_image.png" width="560" height="403"></figure><figure class="image"><img style="aspect-ratio:560/403;" src="1_Docker Install_image.png" width="560" height="403"></figure>
        
        > [!IMPORTANT]
        > If you get an error, you may need to check the module and see if it was updated recently. Incompatibilities in the SQL structure will cause errors.
9.  Repeat steps **6**, **7**, and **8** for each database in each module until all modules are done. Once finished, use `ctrl+c` to stop the containers.

### Configure Server & Modules

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

### Run Server

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

### Game Server

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