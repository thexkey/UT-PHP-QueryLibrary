<?php /** @noinspection SpellCheckingInspection */
/*
 * Unreal Tournament (1999) Server Query Library for PHP
 * A reasonably modern and simple PHP class to query Unreal Tournament (1999) game servers
 * Requires PHP 5.3 or higher. (May work on older versions, but not checked)
 * By thexkey
 *
 * This library is free software;
 * you can redistribute it and/or modify it under the terms of the GNU Lesser General Public License (LGPL) as published by the Free Software Foundation;
 * either version 2.1 of the License, or (at your option) any later version.
 */

// TODO: switch to using socket functions instead of fsockopen


class UTServerQuery
{
    /**
     * The Domain/IP of the server to query information from
     * @var string $ip
     */
    protected $ip;

    /**
     * The port of the server to query information from
     * Default for UT is 7778
     * @var int $port
     */
    protected $port;


    /**
     * Number of seconds to wait for a response from the server before timing out
     * Increase this if you are querying a remote server (or are using a slow connection)
     * Default is 30 seconds
     * @var int $timeout
     */
    protected $timeout;

    /**
     * Shared socket resource for all instances of this class
     * @var resource $socket
     */
    protected $socket = null;


    /**
     * The construction method for the UTServerQuery class
     *
     * @param string $ip The IP or domain of the server to query
     * @param int $port The port of the server to query
     * @param int $timeout The number of seconds to wait for a response from the server before timing out
     *
     * @return void
     */
    public function __construct($ip, $port = 7778, $timeout = 30)
    {
        $this->ip = $ip;
        $this->port = $port;
        $this->timeout = $timeout;
    }

    /**
     * Get the server information
     *
     * @param bool $raw If true, the raw response from the server will be returned instead of a friendly array
     *
     * @return array The server information (hostname, map, game type, players, etc)
     * @throws Exception If the server does not respond with a valid response
     */
    public function getServerInfo($raw = false)
    {
        if ($this->socket === null) {
            // dummy error handling variables
            $errno = 0;
            $errstr = '';

            $this->socket = fsockopen("udp://" . $this->ip ,$this->port ,$errno ,$errstr ,$this->timeout);
            if (!$this->socket) {
                throw new Exception("Could not create socket: $errstr ($errno)");
            }

        }

        // "\\status\\\player_property\Health\\\game_property\ElapsedTime\\\game_property\RemainingTime\\"
        // is the string to send to the server to get the server information
        fwrite($this->socket, "\\status\\\player_property\Health\\\game_property\ElapsedTime\\\game_property\RemainingTime\\");

        $data = '';
        // Loop until final packet has been received.
        do {
            // read the response from the server
            $data .= fread($this->socket, 4096);
        } while (substr($data, -1) != '\\');

        // disconnect from the server
        fclose($this->socket);
        $this->socket = null;

        // convert the response into an array
        // example: \gamename\ut\gamever\469\minnetver\432\location\0\hostname\ROBLOX Reverse-Engineering UT Server -=[VanillaMaps][ACE][Linux]=-\hostport\7777\maptitle\Liandri Central Core\mapname\DM-Liandri\gametype\DeathMatchPlus\numplayers\0\maxplayers\10\gamemode\openplaying\gamever\469\minnetver\432\worldlog\true\wantworldlog\true\listenserver\False\password\False\timelimit\0\fraglimit\30\minplayers\3\changelevels\True\tournament\False\gamestyle\Hardcore\botskill\Average\AdminName\thexkey + Brent\Health_1\100\ElapsedTime\0\RemainingTime\0\queryid\31.1\final
        // should end up as:
        // array(
        //     'gamename' => 'ut',
        //     'gamever' => '469',
        //     'minnetver' => '432',
        //     'location' => '0',
        //     'hostname' => 'ROBLOX Reverse-Engineering UT Server -=[VanillaMaps][ACE][Linux]=-',
        //     'hostport' => '7777',
        //     'maptitle' => 'Liandri Central Core',
        //     'mapname' => 'DM-Liandri',
        //     'gametype' => 'DeathMatchPlus',
        //     'numplayers' => '0',
        //     'maxplayers' => '10',
        //     'gamemode' => 'openplaying',
        //     'gamever' => '469',
        //     'minnetver' => '432',
        //     'worldlog' => 'true',
        //     'wantworldlog' => 'true',
        //     'listenserver' => 'False',
        //     'password' => 'False',
        //     'timelimit' => '0',
        //     'fraglimit' => '30',
        //     'minplayers' => '3',
        //     'changelevels' => 'True',
        //     'tournament' => 'False',
        //     'gamestyle' => 'Hardcore',
        //     'botskill' => 'Average',
        //     'AdminName' => 'thexkey + Brent',
        //     'Health_1' => '100',
        //     'ElapsedTime' => '0',
        //     'RemainingTime' => '0'
        // )

        $data = explode('\\', $data);
        // first array will be empty, so remove it
        array_shift($data);
        // last array will be empty, so remove it
        array_pop($data);

        // convert the array into a key/value array
        $data = array_chunk($data, 2);

        // remove "final" from the array and convert it into a key/value array
        array_pop($data);
        $data = array_combine(array_column($data, 0), array_column($data, 1));

        // if raw response is NOT requested, process the data into more friendly values
        if (!$raw) {
            // if there are any players, get the player information into their own array
            if (isset($data['numplayers']) && $data['numplayers'] > 0) {
                $data['players'] = array();

                /** for every player, a structure like this will be recieved from the server response:
                 * player_(playercount) - player's name
                 * ping_(playercount) - player's network Ping
                 * team_(playercount) - player's team (0 = red, 1 = blue, 2 = green, 3 = yellow, NOTE: spectators do not show up in player list)
                 * frags_(playercount) - player's kills
                 * ngsecret_(playercount) - player's ngWorldStats shared-secret
                 * Health_(playercount +1) - player's health
                 * mesh_(playercount) - player's avatar name
                 * skin_(playercount) - player's avatar file name
                 * face_(playercount) - player's face file name
                **/
                // craft the player array (NOTE: player_ index starts at 0, not 1)
                for($i = 0; $i < $data['numplayers']; $i++) {
                    $data['players'][$i] = array(
                        'name' => $data['player_' . $i],
                        'ping' => $data['ping_' . $i],
                        'team' => $data['team_' . $i],
                        'frags' => $data['frags_' . $i],
                        'ngsecret' => $data['ngsecret_' . $i],
                        'Health' => $data['Health_' . ($i + 2)],
                        'mesh' => $data['mesh_' . $i],
                        'skin' => $data['skin_' . $i],
                        'face' => $data['face_' . $i]
                    );
                    // remove the player information from the main array
                    unset($data['player_' . $i], $data['ping_' . $i], $data['team_' . $i], $data['frags_' . $i], $data['ngsecret_' . $i], $data['Health_' . ($i + 2)], $data['mesh_' . $i], $data['skin_' . $i], $data['face_' . $i]);
                }

            }
        }


        return $data;
    }





}
