Description  
-------------

This plugin is a customizable limits/rules enforcer. It allows you to setup and enforce limits based on player statistics, and server state.   
  
It tracks extensive Battlelog stats, and round stats. If you feel that there is a stat, or aggregate, or information that really needs to be included, post feedback on the PRoCon forums. The plugin supports events like OnKill, OnTeamKill, OnJoin, OnSpawn, etc. You are able to perform actions triggered by those events.  
  
For a full list of examples, jump to the index in the <a href="http://www.phogue.net/forumvb/showthread.php?3448-Insane-Limits-Examples" >Insane Limits - Examples thread.</a>  
  
By default, the plugin ships with **virtual_mode** set to _True_. This allows you to test your limits/rules without any risk of accidentally kicking or banning anyone. Once you feel your limits/rules are ready, you can disable **virtual_mode**.  
   

Twitter Live Feed  
-------------

The live feed shows the plugin state, kicks, and bans for servers that elected to tweet this data.  
  
<a href="http://www.youtube.com/watch?v=TPUqIAx8UOk"><img alt="Twitter Logo" src="http://static.wix.com/media/b96182_4a299d754b010dfdc762290fecc05465.png_128" /></a>

http://twitter.com/InsaneLimits  

  
Video Demo  
-------------

Simple video demo/tutorial showcasing the how to use the plugin.

<a href="http://www.youtube.com/watch?v=TPUqIAx8UOk"><img alt="Insane Limits PRoCon Plugin (BF3)" src="http://i1.ytimg.com/vi/TPUqIAx8UOk/default.jpg" /></a>


Disclaimer  
-------------

If you are careless with the rules, sometimes you can end up hurting more than helping. Having said that, it's not up to me or anyone to judge the merit of a limit/rule, otherwise this can become a flame war. Be polite when replying to posts, and don't flame others because you don't like their rules. I just give you the tool, and you (the server admin) make the rules.  
  
My advise is, think creatively when coming up with the rules, and be open minded. If you do not know how to express your limit ideas in C#, ask the PRoCon forums. The data is there, available, ready to be consumed ... you just have to come up with the ideas.  
 

Installation Instructions  
-------------

Download the zip file containing the plugin source (in the attachments) Extract the source file InsaneLimits.cs Copy the source file to ProCon's Plugins/BF3 directory  


Minimum Requirements  
-------------

This plugin requires you to have sufficient privileges for running the following commands:  
  
* serverInfo
* mapList.list
* mapList.getMapIndices
* admin.listPlayers all
* punkBuster.pb_sv_command pb_sv_plist
* punkBuster.pb_sv_command pb_sv_ban
* punkBuster.pb_sv_command pb_sv_kick
  
Additionaly, you need to have Read+Write file system permission in the following directories:   
  

 * &lt;ProCon&gt;/
 * &lt;ProCon&gt;/Plugins/BF3


  
Supported Limit Evaluations  
-------------

* **OnJoin** - Limit evaluated when player joins 
* **OnLeave** - Limit evaluated when player leaves
* **OnSpawn** - Limit evaluated when player spawns
* **OnKill** - Limit evaluated when makes a kill (team-kills not counted)
* **OnTeamKill** - Limit evaluated when player makes a team-kill
* **OnDeath** - Limit evaluated when player dies (suicides not counted)
* **OnTeamDeath** - Limit evaluated when player is team-killed
* **OnSuicide** - Limit evaluated when player commits suicide
* **OnAnyChat** - Limit evaluated when players sends a chat message
* **OnInterval** - (deprecated) Same behavior as **OnIntervalPlayers**
* **OnIntervalPlayers** - Limit evaluated (for all players) every **evaluation_interval** number of seconds
* **OnIntervalServer** - Limit evaluated once every **evaluation_interval** number of seconds
* **OnRoundOver** - Limit evaluated when round over event is sent by PRoCon
* **OnRoundStart** - Limit evaluated after round over event, when first player 
* **OnTeamChange** - Limit evaluated after player switches team (admin move not counted as team-switch)
  
> Note that limit evaluation is only performed after the plugin has fetched the player stats from Battlelog.   
  
When you enable the plugin for the first time in a full server, it will take a couple of minutes to fetch all player stats  
  
Architecture  
-------------
  
When the plugin is enabled, it starts two threads:   
  
The **fetch** thread is in charge of monitoring the players that join the server. It fetches player statistics from battlelog.battlefield.com  
  
The **enforcer** thread is in charge of evaluating Interval limits. When the **enforcer** thread finds that a player violates a limit, it performs an action (Kick, Ban, etc) against that player.  
  
The two threads have different responsibilities, but they synchronize their work.   
  
  
Fetch-thread Flow  
-------------
  
When players join the server, they are added the stats queue. The fetch thread is constantly monitoring this queue. If there is a player in the queue, it removes him from the queue, and fetches the battlelog stats for the player.  
  
The stats queue can grow or shrink depending on how fast players join, and how long the web-requests take. If you enable the plugin on a full server, you will see that almost immediately all players are queued for stats fetching. Once the stats are fetched for all players in the queue, they are added to the internal player's list.  
  
  
Enforcer-thread Flow  
-------------

The enforcer thread runs on a timer (every second). It checks if there are any interval limits ready to be executed. If there are, it will evaluate those limits.  
  
Each time around that the **enforcer** checks for the available limits is called an _iteration_. If there are no players in the server, or there are no limits available, the **enforcer** skips the current _iteration_ and sleeps until the next _iteration_.  
  
The enforcer is only responsible for Limits that evaluate OnIterval, events. Enforcing for other types of events like OnKill, and OnSpawn, is done in the main thread when procon sends the event information.   
  
  
Limit Management  
-------------
  
**Creation** - In order to create a new limit, you have to set **new_limit** variable to **True**.  
  
> This creates a new limit section with default values that you can change.  
  
  
**Deletion** - In order to delete a limit, you have to set the variable **delete_limit** to the numerical _id_ of the limit you want to delete.  
  
> Each limit has an **id** number, you can see the **id** number in the limit name, e.g. Limit #**5**.  
  
Limit Definition  
-------------
  
At its basic, there are four fields that determine the structure of a limit. These fields are **state**, **action**, and **first_check**, and **second_check**.  
  
**state**  
* _Enabled_- the limit will be used by the plugin thread   
* _Disabled_ - the limit will be ignored by the plugin thread  
* _Virtual_ - the limit will be used, but actions will be done in **virtual_mode**  
  
This field is useful if you want to temporarily disable a limit from being used, but still want to preserve its definition.  
  
**action**  
* _(string, psv)_ - list of actions for this limit (Pipe separated ""|"")  
  
e.g. Say | PBBan | Mail   
  
These are all the allowed actions:  

* _None_ - no action is performed against the player
* _Kick_ - player is kicked, if the limit evaluates to _True_
* _EABan_ - player is banned (using the BF3 ban-list), if the limit evaluates _True_
* _PBBan_ - player is banned (using PunkBuster ban-list), if the limit evaluates _True_
* _Kill_ - kills the player (delay optional), if the limit evaluates _True_
* _Say_ - sends a message the server (All, Team, Squad, or Player), if the limit evaluates _True_
* _Log_- logs a message to a File, Plugin log, or both, if the limit evaluates _True_
* _Mail_ - sends an e-mail to specified address, if the limit evaluates _True_
* _SMS_ - sends an SMS message to the specified phone number, if the limit evaluates _True_
* _Tweet_ - posts a Twitter status update (default account is @InsaneLimits), if the limit evaluates _True_
* _PBCommand_ - executes the specified PunkBuster command, if the limit evaluates _True_
* _ServerCommand_ - executes the specified Server command, if the limit evaluates _True_
* _PRoConChat_ - sends the specified text to PRoCon's Chat-Tab, if the limit evaluates _True_
* _PRoConEvent_ - adds the specified event to PRoCon's Events-Tab, if the limit evaluates_True_
* _TaskbarNotify_ - sends a Windows Taskbar notification, if the limit evaluates _True_
  
Depending on the selected action, other fields are shown to specify more information about the action.  
  
> Supported PB ban-duration: _Permanent_, _Temporary_  
> Supported PB ban-type: _PB_GUID_ (default)  
  
> Supported EA ban-duration: _Permanent_, _Temporary_, _Round_  
> Supported EA ban-type: _EA_GUID_, _IPAddress_, _Name_  
  
Also note that each of these actions have a **target** player. You have to be careful on what **target** is for each action.  
  
For example, during a Kill event, the target of the action is the Killer. But, during a Death event, the target of the action is the player that was killed. You don't want to accidentally Kick/Ban the wrong player!  
  
* **first_check**
    * _Disabled_ - the limit does not evaluate anything in the first step of evaluation  
    * _Expression_ - the limit uses a C# conditional expression during the first step of evaluation  
    * _Code_ - the limit uses a C# code snippet (must return true/false) during the first step of evaluation  
  
* **second_check**  
    * _Disabled_ - the limit does not evaluate anything in the second step of evaluation  
    * _Expression_ - the limit uses a C# conditional expression during the second step of evaluation  
    * _Code_ - the limit uses a C# code snippet (must return true/false) during the second step of evaluation  
  
Depending on the selected check type, an extra field will be shown for specifying the _Expression_, or _Code_ text.  
Both _Expressions_, and _Code_ snippets must be syntactically correct in accordance to the C# language.The plugin compiles your _Expression_/_Code_ in-memory with the Microsoft C# Compiler. If there are compilation errors, those are shown in the plugin log.  
  
If you do not know what C# is, or what an expression is, or what a code snippet is ... do not worry. Study the examples in the <a href="http://www.phogue.net/forumvb/showthread.php?3448-Insane-Limits-Examples" >Insane Limits - Examples thread.</a>. Then, if you are still unclear, how to write an expression or a code snippet, ask for help in the PRoCon forums.  
  
Limit Evaluation  
-------------
  
After compilation, limit evaluation is by far the most important of all steps this plugin goes through.  
  
Limit evaluation is comprised of three steps:
  
1. **first_check** Evaluation

  During this step, the plugin executes the _Expression_/_Code_ in **first_check** to get a _True_ or _False_ result.  

  If the result is _False_ , the plugin does not perform any **action**, and quits. But, if it's _True_ , it keeps going to the next step   
  
2. **second_check** Evaluation (optional)
  
  Next, the plugin runs the _Expression_/_Code_ for the **second_check**, if it's enabled. If it's not enabled, it keeps going to next step.  
  
3. **action** Execution

  If the final result of the limit evaluation is _True_, the plugin then executes the **action** associated with the limit.  
  
  If the final result of the limit evaluation is _False_, no **action** is executed.  


Objects  
-------------

When writing a limit _Expression_ or _Code_ snippet, there are several globally defined objects that can be used. These are **server**, **player**, **killer**, **victim**, **kill**, **plugin**, **team1**, **team2**, **team3**, **team4**, and **limit**. These objects contain values, and functions that can be accessed from within the _Expressions_, or _Code_ snippets.  
  

Limit Object  
-------------
  
The **limit** object represents the state the limit that was just activated. This object is only available during the **second_check**. The **limit** object implements the following interface:  
  
~~~
public interface LimitInfoInterface
{
   //Number of times the limit has been activated, (Current round)
    double Activations(String PlayerName);
    double Activations(int TeamId, int SquadId);
    double Activations(int TeamId);

    // Number of times player has activated this limit (Current round) in the given TimeSpan, e.g. last 10 seconds, etc
    double Activations(String PlayerName, TimeSpan time);
    double Activations();


    //Number of times the limit has been activated (All rounds)
    double ActivationsTotal(String PlayerName);
    double ActivationsTotal(int TeamId, int SquadId);
    double ActivationsTotal(int TeamId);
    double ActivationsTotal();
   
    // Number of times this limit has been activated by player
    /* 
     * Kill, TeamKill: Spree value is reset when player dies
     * Death, TeamDeath, and Suicide: Spree value is reset whe player makes a kill
     * 
     * Spawn, Join, Interval: Spree value is never reset, you may reset it manually.
     */
    
    double Spree(String PlayerName);

    // manually resets the Spree value for the player, (only for power-users)
    void ResetSpree(String PlayerName);
    
    /* Data Repository set/get custom data */
    DataDictionaryInterface Data { get; }
    DataDictionaryInterface RoundData { get; }   //this dictionary is automaticaly cleared OnRoundStart
    
    /* Other methods */
    String LogFile { get; }
}
~~~

Team Object (team1, team2, team3, team4)  
-------------
  
The **teamX** object represents the state of the team with id X at the moment that the limit is being evaluated. The **teamX** object implements the following interface:  
  
~~~
public interface TeamInfoInterface
{
    List<PlayerInfoInterface> players { get; }

    double KillsRound { get; }
    double DeathsRound { get; }
    double SuicidesRound { get; }
    double TeamKillsRound { get; }
    double TeamDeathsRound { get; }
    double HeadshotsRound { get; }
    double ScoreRound { get; }

    int TeamId { get; }
    double Tickets { get; }      
    double RemainTickets { get; }
    double RemainTicketsPercent { get; }
    double StartTickets { get; }
}
~~~

Server Object  
-------------

The **server** object represents the state of the server at the moment that the limit is being evaluated. The **server** object implements the following interface:  
  
~~~
public interface ServerInfoInterface
{
    /* Server State */
    int CurrentRound { get; }
    int TotalRounds { get; }
    int PlayerCount { get; }
    int MaxPlayers { get; }
    
    /* Current Map Data */
    int MapIndex { get; }
    String MapFileName { get; }
    String Gamemode { get; }

    /* Next Map Data */
    int NextMapIndex { get; }
    String NextMapFileName { get; }
    String NextGamemode { get; }

    /* All players, Current Round, Stats */
    double KillsRound { get; }
    double DeathsRound { get; }   // kind of useless, should be same as KillsTotal (suicides not counted as death)
    double HeadshotsRound { get; }
    double SuicidesRound { get; }
    double TeamKillsRound { get; }
    
    /* All players, All rounds, Stats */
    double KillsTotal { get; }
    double DeathsTotal { get; }  // kind of useless, should be same s KillsTotal (suicides not counted as death)
    double HeadshotsTotal { get; }
    double SuicidesTotal { get; }
    double TeamKillsTotal { get; }

    /* Weapon Stats, Current Round, All Rounds (Total)*/
    WeaponStatsInterface this[String WeaponName] { get; }
    
    /* Other data */
    double TimeRound { get; }                // Time since round started
    double TimeTotal { get; }                // Time since plugin enabled 
    double TimeUp { get; }                   // Time since last server restart
    double RoundsTotal { get; }              //Round played since plugin enabled
    
    /* Meta Data */
    String Port { get; }                     // Layer/Server port number
    String Host { get; }                     // Layer/Server Host
    String Name { get; }
    String Description { get; }

    /* Team data */
    double Tickets(int TeamId);              //tickets for the specified team
    double RemainTickets(int TeamId);        //tickets remaining on specified team
    double RemainTicketsPercent(int TeamId); //tickets remaining on specified team (as percent)

    double StartTickets(int TeamId);         //tickets at the begining of round for specified team
    double TargetTickets { get; }            //tickets needed to win
    
    int OppositeTeamId(int TeamId);          //id of the opposite team, 1->2, 2->1, 3->4, 4->3, *->0
    int WinTeamId { get; }                   //id of the team that won previous round
    
    /* Data Repository set/get custom data */
    DataDictionaryInterface Data { get; }
    DataDictionaryInterface RoundData { get; }   //this dictionary is automaticaly cleared OnRoundStart
}
~~~

Kill Object  
-------------
  
The **kill** object represents information about the kill event. The **kill** object implements the following interface:  
  
~~~
public interface KillInfoInterface
{
    String Weapon { get; }
    bool Headshot { get; }
    DateTime Time { get; }
}
~~~

Player, Killer, Victim Objects  
-------------
  
The **player** object represents the state of player for which the current limit is being evaluated. The **player** object implements the following interface:  
  
~~~
public interface PlayerInfoInterface
{
    /* Online statistics (battlelog.battlefield.com) */
    double Rank { get; }
    double Kdr { get; }
    double Time { get; }
    double Kills { get; }
    double Wins { get; }
    double Skill { get; }
    double Spm { get; }
    double Score { get; }
    double Deaths { get; }
    double Losses { get; }
    double Repairs { get; }
    double Revives { get; }
    double Accuracy { get; }
    double Ressuplies { get; } 
    double QuitPercent { get; }
    double ScoreTeam { get; }
    double ScoreCombat{ get; }
    double ScoreVehicle{ get; }
    double ScoreObjective { get; }
    double VehiclesKilled { get; }
    double Kpm { get; }

    double ReconTime { get; }
    double EngineerTime { get; }
    double AssaultTime { get; }
    double SupportTime { get; }
    double VehicleTime { get; }
    double ReconPercent { get; }
    double EngineerPercent { get; }
    double AssaultPercent { get; }
    double SupportPercent { get; }
    double VehiclePercent { get; }

    /* Player data */
    String Name { get; }
    String FullName { get; } // name including clan-tag
    String Tag { get; }
    String IPAddress { get; }
    String CountryCode { get ; } 
    String CountryName { get; }
    String PBGuid { get; }
    String EAGuid { get; }
    int TeamId { get; }
    int SquadId { get; }

    
    /* Current round, Player Stats */
    double KdrRound { get; }
    double KpmRound { get; }
    double SpmRound { get; }
    double ScoreRound { get; }
    double KillsRound { get; }
    double DeathsRound { get; }
    double HeadshotsRound { get; }
    double TeamKillsRound { get; }
    double TeamDeathsRound { get; }
    double SuicidesRound { get; }
    double TimeRound { get; }

    /* All rounds, Player Stats */
    double KdrTotal { get; }
    double KpmTotal { get; }
    double SpmTotal { get; }
    double ScoreTotal { get; }
    double KillsTotal { get; }
    double DeathsTotal { get; }
    double HeadshotsTotal { get; }
    double TeamKillsTotal { get; }
    double TeamDeathsTotal { get; }
    double SuicidesTotal { get; }
    double TimeTotal { get; }
    double RoundsTotal { get; }

    /* Weapon Stats, Current Round, All Rounds (Total) */
    WeaponStatsInterface this[String WeaponName] { get; }

    /* Other Data */
    DateTime JoinTime { get; }
    String LastChat { get; }   // text of the last chat sent by player
    bool Battlelog404 { get; } // True - Player has no PC Battlelog profile
    bool StatsError { get; }   // True - Error occurred while processing player stats
    
    /* Whitelist information */
    bool inClanWhitelist { get; }
    bool inPlayerWhitelist { get; }
    bool isInWhitelist { get; }
    
    /* Data Repository set/get custom data */
    DataDictionaryInterface Data { get; }
    DataDictionaryInterface RoundData { get; }   //this dictionary is automaticaly cleared OnRoundStart
}
~~~

Plugin Object  
-------------

The **plugin** represents this plugin itself. It gives you access to important functions for executing server commands, and interacting with ProCon. The **plugin** object implements the following interface:  
  
~~~
public interface PluginInterface
{
    /*
     * Methods for sending messages 
     */
    bool SendGlobalMessage(String message);
    bool SendTeamMessage(int teamId, String message);
    bool SendSquadMessage(int teamId, int squadId, String message);

    bool SendGlobalMessage(String message, int delay);
    bool SendTeamMessage(int teamId, String message, int delay);
    bool SendSquadMessage(int teamId, int squadId, String message, int delay);

    bool SendMail(String address, String subject, String body);
    bool SendSMS(String country, String carrier, String number, String message);

    /*
     * Methods used for writing to the Plugin console
     */
    void ConsoleWrite(String text);
    void ConsoleWarn(String text);
    void ConsoleError(String text);
    void ConsoleException(String text);

    /*
     * Methods for getting whitelist information 
     * 
     */
    bool isInWhitelist(String PlayerName);
    bool isInPlayerWhitelist(String PlayerName);
    bool isInClanWhitelist(String PlayerName);
    bool isInWhiteList(String PlayerName, String list_name);

    /*
     * Methods getting and setting the Plugin's variables
     */
    bool setPluginVarValue(String variable, String value);
    String getPluginVarValue(String variable);

    /*
     *  Method: R
     *  
     *  Replaces tags like %p_n% (Player Name), %k_n% (Killer Name), %v_n% (Victim Name), etc          
     */
    String R(String message);

    /* 
     * Methods for actions
     */

    bool KickPlayerWithMessage(String name, String message);
    bool KillPlayer(String name);  /* deprecated, do not use */
    bool KillPlayer(String name, int delay);
    bool EABanPlayerWithMessage(EABanType type, EABanDuration duration, String name, int minutes, String message);
    bool PBBanPlayerWithMessage(PBBanDuration duration, String name, int minutes, String message);
    bool PBCommand(String text);
    bool MovePlayer(String name, int TeamId, int SquadId, bool force);
    bool SendTaskbarNotification(String title, String messagge);
    bool Log(String file, String message);
    bool Tweet(String status);
    bool PRoConChat(String text);
    bool PRoConEvent(String text, String player);

    void ServerCommand(params String[] arguments);

    /* Other Methods */
    String FriendlySpan(TimeSpan span);       //converts a TimeSpan into a friendly formatted string e.g. "2 hours, 20 minutes, 15 seconds"
    String BestPlayerMatch(String name);      //looks in the internal player's list, and finds the best match for the given player name
    
    bool IsInGameCommand(String text);          //checks if the given text start with one of these characters: !/@?
    bool IsCommand(String text);                //checks if the given text start with one of these characters: !/@?

    String ExtractInGameCommand(String text);   //if given text starts with one of these charactets !/@? it removes them
    String ExtractCommand(String text);         //if given text starts with one of these charactets !/@? it removes them

    String ExtractCommandPrefix(String text);   //if given text starts with one of these chracters !/@? it returns the character

    /*
     * Creates a file in ProCOn's directory  (InsaneLimits.dump)
     * Detailed information about the exception.
     */
    void DumpException(Exception e);
    
    /* Data Repository set/get custom data */
    DataDictionaryInterface Data { get; }
    DataDictionaryInterface RoundData { get; }   //this dictionary is automaticaly cleared OnRoundStart
}
~~~

Data and RoundData Objects  
-------------
  
The **Data** object is a nested dictionary of key/value pairs that you can use to store custom data inside the **plugin**, **server**, **limit**, **player**, **killer**, and **victim** objects. The **Data** object implements the following interface:   
  
~~~
public interface DataDictionaryInterface
{
    /* String Data */
    String setString(String key, String value);
    String getString(String key);
    String unsetString(String key);
    bool issetString(String key);
    List<String> getStringKeys();

    /* Boolean Data */
    bool setBool(String key, bool value);
    bool getBool(String key);
    bool unsetBool(String key);
    bool issetBool(String key);
    List<String> getBoolKeys();
    
    /* Double Data */
    double setDouble(String key, double value);
    double getDouble(String key);
    double unsetDouble(String key);
    bool issetDouble(String key);
    List<String> getDoubleKeys();
    
    /* Int Data */
    int setInt(String key, int value);
    int getInt(String key);
    int unsetInt(String key);
    bool issetInt(String key);
    List<String> getIntKeys();
    
    /* Object Data */
    object setObject(String key, object value);
    object getObject(String key);
    object unsetObject(String key);
    bool issetObject(String key);
    List<String> getObjectKeys();
    
    /* Generic set/get methods */
    Object set(Type type, String key, Object value);
    Object get(Type type, String key);
    Object unset(Type type, String key);
    bool isset(Type type, String key);
    List<String> getKeys(Type type);

    /* Other methods */
    void Clear();  /* clear/unset all data from repository */
    
}
~~~

Simple (Traditional) Replacements  
-------------
  
This plugin supports an extensive list of message text replacements. A replacement is a string that starts and ends with the percent character %. When you use them in the text of a message, the plugin will try to replace it with the corresponding value. For example:   
  
The message
~~~
    %k_n% killed %v_n% with a %w_n%
~~~

becomes

~~~
    micovery killed NorthEye with a PP-2000
~~~

Below is the list of all supported simple replacements.
~~~
    // Killer Replacements (Evaluations:  OnKill, OnDeath, OnTeamKills, and OnTeamDeath)

    /* Legend:
     * k   - killer
     * n   - name
     * ct  - Clan-Tag
     * cn  - Country Name
     * cc  - Country Code
     * ip  - IPAddress
     * eg  - EA GUID
     * pg  - Punk Buster GUID
     */
     
    %k_n%    Killer name
    %k_ct%   Killer clan-Tag
    %k_cn%   Killer county-name
    %k_cc%   Killer county-code
    %k_ip%   Killer ip-address
    %k_eg%   Killer EA GUID
    %k_pg%   Killer Punk-Buster GUID
    %k_fn%   Killer full name, includes Clan-Tag (if any)

    // Victim Replacements (Evaluations:  OnKill, OnDeath, OnTeamKills, and OnTeamDeath)

    /* Legend:
     * v   - victim
     */
     
    %v_n%    Victim name,
    %v_ct%   Victim clan-Tag
    %v_cn%   Victim county-name
    %v_cc%   Victim county-code
    %v_ip%   Victim ip-address
    %v_eg%   Victim EA GUID
    %v_pg%   Vitim Punk-Buster GUID
    %v_fn%   Victim full name, includes Clan-Tag (if any)

    // Player Repalcements (Evaluations: OnJoin, OnSpawn, OnAnyChat, OnTeamChange, and OnSuicide)

    /* Legend:
     * p   - player
     * lc  - last chat
     */
     
    %p_n%    Player name
    %p_ct%   Player clan-Tag
    %p_cn%   Player county-name
    %p_cc%   Player county-code
    %p_ip%   Player ip-address
    %p_eg%   Player EA GUID
    %p_pg%   Player Punk-Buster GUID
    %p_fn%   Player full name, includes Clan-Tag (if any)
    %p_lc%   Player, Text of last chat

    // Weapon Replacements (Evaluations: OnKill, OnDeath, OnTeamKill, OnTeamDeath, OnSuicide)

    /* Legend:
     * w   - weapon
     * n   - name
     * p   - player
     * a   - All (players)
     * x   - count
     */
     
    %w_n%    Weapon name,
    %w_p_x%  Weapon, number of times used by player in current round
    %w_a_x%  Weapon, number of times used by All players in current round

    // Limit Replacements for Activations & Spree Counts, Current Round (Evaluations: Any)

    /* Legend:
     * th  - ordinal count suffix e.g. 1st, 2nd, 3rd, 4th, etc
     * x   - count, 1, 2, 3, 4, etc
     * p   - player
     * s   - squad
     * t   - team
     * a   - All (players)
     * r   - SpRee
     */
     
    %p_x_th%  Limit, ordinal number of times limit has been activated by the player
    %s_x_th%  Limit, ordinal number of times limit has been activated by the player's squad
    %t_x_th%  Limit, ordinal number of times limit has been activated by the player's team
    %a_x_th%  Limit, ordinal number of times limit has been activated by all players in the server
    %r_x_th%  Limit, ordinal number of times limit has been activated by player without Spree value being reset
    %p_x%     Limit, number of times limit has been activated by the player
    %s_x%     Limit, number of times limit has been activated by the player's squad
    %t_x%     Limit, number of times limit has been activated by the player's team
    %a_x%     Limit, number of times limit has been activated by all players in the server
    %r_x%     Limit, number of times limit has been activated by player without Spree value being reset
    
    // Limit Replacements for Activations, All Round (Evaluations: Any) 
    /* Legend:
     * xa - Total count, for all rounds
     */
    %p_xa_th%  Limit, ordinal number of times limit has been activated by the player
    %s_xa_th%  Limit, ordinal number of times limit has been activated by the player's squad
    %t_xa_th%  Limit, ordinal number of times limit has been activated by the player's team
    %a_xa_th%  Limit, ordinal number of times limit has been activated by all players in the server
    %p_xa%     Limit, number of times limit has been activated by the player
    %s_xa%     Limit, number of times limit has been activated by the player's squad
    %t_xa%     Limit, number of times limit has been activated by the player's team
    %a_xa%     Limit, number of times limit has been activated by all players in the server

    // Other replacements
    %date%    Current date, e.g. Sunday December 25, 2011,
    %time%    Current time, e.g. 12:00 AM
    
    %server_host%  Server/Layer host/IP
    %server_port%  Server/Layer port number
    
    %l_id% Limit numeric id"
    %l_n%  Limit name""

~~~

Advanced Replacements  
-------------

In addition to the simple %**key**% replacments, this plugin also allows you to use a more advanced type of replacement. Within strings, you can use replacements that match properties in known objects. For example, if you use **player.Name** within a string, the plugin will detect it and replace it appropriately.  
  
A common usage for advanced replacements is to list player stats in the Kick/Ban reason. For example:  
  
The message   
  
~~~
    player.Name you were banned for suspicious stats: Kpm: player.Kpm, Spm: player.Spm, Kdr: player.Kdr
~~~

becomes

~~~
    micovery you were banned for suspicious stats: Kpm: 0.4, Spm: 120, Kdr: 0.61
~~~

Settings  
-------------
  
0. **player_white_list**

  _(string, csv)_ - list of players that should never be kicked or banned
  
0. **clan_white_list**

  _(string, csv)_ - list of clan (tags) for players that should never be kicked or banned
  
0. **virtual_mode**

  _true_ - limit **actions** (kick, ban) are simulated, the actual commands are not sent to server
  _false_ - limit **actions** (kick, ban) are not simulated
  
0. **console**

  _(string)_ - you can use this field to run plugin commands
  
> For example: !stats micovery will print the player statistic for the current round in the plugin console.
  
Note that plugin commands, are currently supported only inside ProCon, and not In-Game. 
  
0. **smtp_port**

  _(String)_ - Address of the SMTP Mail server used for _Mail_ action
  
0. **smtp_port**

  _(integer &gt; 0)_ - port number of the SMTP Mail server used for _Mail_ action  
  
0. **smtp_account**

  _(Stirng)_ - mail address for authenticating with the SMTP Mail used for _Mail_ action  
  
0. **smtp_mail**

  _(Stirng)_ - mail address (Sender/From) that is used for sending used for _Mail_ action  
  
  > This is usually the same as **smtp_account** ... depends on your SMTP Mail provider.
  
0. **smtp_ssl**

  _true_ - mail sent using secure socket (use this only if your SMTP provider requires it)  
  _false_ - mail sent without using secure socket  
  
0. **say_interval**

  _(float)_ - interval in seconds between say messages. Default value is 0.05, which is 50 milli-seconds  
  
  > The point of this setting is to avoid spam, but you should not set this value too large. Ideally it should be between 0 and 1 second.  
  
0. **auto_hide_sections**

  _true_ - when creating a new section, it will be hidden by default  
  _false_ - when creating a new section, it will not be hidden  
  
0. **twitter_reset_defaults**

  _true_ - resets the Twitter's **access_token**, and **access_token_secret** back to default values  
  
0. **twitter_setup**

  _true_ - initiates the Twitter configuration, it will show a link that you have to visit to get the verification PIN  
  
0. **twitter_verifier_pin**

  _(String)_ - this field is displayed for you to enter the Twitter verification PIN, it disappears as soon as you enter it
  
  > After entering your PIN, the plugin will try to exchange the PIN for a Twitter **access_token**, and **access_token_secret**. If the verifcation stage fails, you must re-initiate the Twitter setup process.   
  
  
Plugin Commands  
-------------
  
These are the commands supported by this plugin. You can run them from within the **console** field. Replies to the commands are printed in the plugin log.  
  
* 
  **!round stats**  
  Aggregate stats for all players, current round  
  
  **!total stats**  
  Aggregate stats for all players, all rounds  
  
  **!weapon round stats**  
  Weapon-Level aggregate stats for all players, current round  
  
  **!weapon total stats**  
  Weapon-Level aggregate stats for all players, all rounds  
  
  **!web stats {player}**  
  Battlelog stats for the current player  
  
  **!round stats {player}**  
  Aggregate stats for the current player, current round  
  
  **!total stats {player}**  
  Aggregate stats for the current player, all rounds  
  
  **!weapon round stats {player}**  
  Weapon-Level stats for the current player, current round  
  
  **!weapon total stats {player}**  
  Weapon-Level stats for the current player, all round  

  
  These are the most awesome of all the commands this plugin provides. Even if you are not using this plugin to enforce any limit, you could have it enabled for just monitoring player stats.  
  
  When calling player specific statistic commands, if you misspell, or only type part of the player name, the plugin will try to find the best match for the player name.  
  
* **!dump limit {id}**  
  
  This command creates a file in ProCon's directory containing the source-code for the limit with the specified _id_  
  
  For example, the following command   

  > !dump limit **5**  
  
  Creates the file LimitEvaluator**5**.cs inside ProCon's directory.   
  
  This command is very useful for debugging compilation errors, as you can see the code inside the file exactly as the plugin sees it (with the same line and column offsets).  
  
* 
  **!set {variable} {to|=} {value}**  
  **!set {variable} {value}**   
  **!set {variable}**  
  
  This command is used for setting the value of this plugin's variables.  
  For the last invocation syntax the value is assumed to be True.   
  
* **!get {variable} **  
  
  This command prints the value of the specified variable.  
  

In-Game Commands  
-------------
  
These are the In-Game commands supported by this plugin. You can run them only from within the game. Replies to the commands are printed in the game chat.  
  

* **!stats**  
  List the available stats, Battlelog  
  
* **!stats [web|battlelog]**  
  List the available stats, Battlelog  
  
* **!stats round**  
  List the available stats, current round  
  
* **!stats total**  
  List the available stats, all rounds  

These commands are used as a shortcut for players to view what type of stats they can query. The plugin will try to fit all stat types into a single chat message.  
  
* **!my {type}**  
  Print Battlelog stat of the specified **type** for the player that executed the command  
  
* **!my round {type}**  
  Print current round stat of the specified **type** for the player that executed the command  
  
* **!my total {type}**  
  Print all rounds stat of the specified **type** for the player that executed the command  
  
* **?{player} {type}**  
  Print Battlelog stat of the specified **type** for the specified **player**  
  
* **?{player} round {type}**  
  Print current round stat of the specified **type** for the specified **player**  
  
* **?{player} total {type}**  
  Print all rounds stat of the specified **type** for the specified **player**  
  
  The **player** name can be a sub-string, or even misspelled. The plugin will find the best match.  
  
  These are the allowed stat **types**:

  > **Battlelog**: kdr, kpm, time, kills, wins, skill, spm, score, deaths, losses, repairs, accuracy, quit pecent, team score, combat socre, objective score, vehicles killed, recon time, engineer time, assault time, support time, vehicle time, engineer percent, assault percent, vehicle percent 
  
  > **Round**: kdr, kpm, spm, score, kills, deaths, headshots, team kills, team deaths, suicides, time Total: kdr, kpm spm, socre, kills, deaths, headshots, team kills, team deaths, suicides, time, rounds
  
  Here are some example commands:  

~~~
    !my kdr  
    !my spm  
    !my engineer percent  
    !micovery recon time  
~~~  
  
Wish-List for 0.0.0.9 (in order of priority)
-------------

- Enable LINQ support for Battlelog weapon stats 
- Add replacements for all custom dictionary keys in player, killer, victim, limit, server, plugin object  

~~~
    " key1 = player.DataRound.String(key1), key2 = plugin.Data.Int(key2)" 
~~~

- Add option to compile limit from class file 
- Add option to specify In-Game command prefixes 
- Enable weapon-level In-Game stat commands 
- Add option to get player's ping  