#!/usr/bin/env python2

# Copyright (c) <2012>, <Malcolm Still a.k.a. klltkr>
# All rights reserved.
# 
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are met:
#     * Redistributions of source code must retain the above copyright
#       notice, this list of conditions and the following disclaimer.
#     * Redistributions in binary form must reproduce the above copyright
#       notice, this list of conditions and the following disclaimer in the
#       documentation and/or other materials provided with the distribution.
#     * Neither the name of the <organization> nor the
#       names of its contributors may be used to endorse or promote products
#       derived from this software without specific prior written permission.
# 
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
# ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
# WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
# DISCLAIMED. IN NO EVENT SHALL <COPYRIGHT HOLDER> BE LIABLE FOR ANY
# DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
# (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
# LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
# ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
# (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
# SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

import curses
import os
import time
import locale
import subprocess
import collections
import sys
import signal

from Queue import Queue, Empty
from threading import Thread
from gmusicapi.api import Api
from getpass import getpass
from operator import itemgetter

def parse_config():
    defaultcolors = { 'thunner':'default',
                      'header-text':'default',
                      'header-sep':'default',
                      'header-border':'default',
                      'footer-border':'default',
                      'footer-sep':'default',
                      'status':'default',
                      'current-artist':'default',
                      'current-song':'default',
                      'current-item':'default',
                      'text':'default' }
    config = { 'colors':[], 'assignments':defaultcolors }
    try:
        with open(os.path.expanduser("~/.thunnerrc")) as f:
            lines = f.readlines()
            for i in lines:
                words = i.split()
                if words[0] == 'email':
                    config['email'] = words[1]
                elif words[0] == 'pass':
                    config['pass'] = i.split(' ',1)[1]
                elif words[0] == 'color':
                    config['colors'].append((words[1],words[2],words[3]))
                elif words[0] == 'assign':
                    config['assignments'][words[2]] = words[1]
            if 'pass' not in config:
		    # If Password not in Config, ask for it
		    config['pass'] = getpass()
    except IOError:
        print "Can't find ~/.thunnerrc"
        sys.exit()
    return config

def color(c):
    if c == 'default':
        return -1
    elif c == 'black':
        return curses.COLOR_BLACK
    elif c == 'red':
        return curses.COLOR_RED
    elif c == 'green':
        return curses.COLOR_GREEN
    elif c == 'yellow':
        return curses.COLOR_YELLOW
    elif c == 'blue':
        return curses.COLOR_BLUE
    elif c == 'magenta':
        return curses.COLOR_MAGENTA
    elif c == 'cyan':
        return curses.COLOR_CYAN
    elif c == 'white':
        return curses.COLOR_WHITE

def name_colors(colordefs):
    colormap = { 'default':1 }
    curses.init_pair(1,-1,-1)
    c = 2
    for i in colordefs:
        curses.init_pair(c,color(i[1]),color(i[2]))
        colormap[i[0]] = c
        c += 1
    return colormap

def apiinit(email,password):
    api = Api()
    
    logged_in = False
    attempts = 0
    api.login(email, password)
    return api

def scrinit():
    stdscr = curses.initscr()
    curses.start_color()
    curses.noecho()
    curses.cbreak()
    curses.curs_set(0)
    stdscr.keypad(1)
    return stdscr

def scrrelease(s):
    curses.nocbreak()
    s.keypad(0)
    curses.echo()
    curses.endwin()

def drawstring(scr,y,x,width,cslist,code,colors,colormap):
    for i in cslist:
        s = i[0]
        col = i[1]
        if (width - x) > 0:
            scr.addnstr(y,x,s.encode(code),width-x,curses.color_pair(colors[colormap[col]]))
        x += len(s)

def drawline(scr,y,width,slen,cs):
    s = cs[0]
    col = cs[1]
    scr.addnstr(y,0,s,width,curses.color_pair(col))
    for i in range(slen,width):
        scr.addstr(y,slen,' ',curses.color_pair(col))
        slen += 1
    
def header(scr,width,parents,current,code,colors,colormap):
    locstring = [('thunner','thunner')]
    for i in parents[1:]:
        locstring.append((' > ','header-sep'))
        locstring.append((i['name'],'header-text'))
    if parents:
        locstring.append((' > ','header-sep'))
        locstring.append((current['name'],'header-text'))
    drawstring(scr,0,0,width,locstring,code,colors,colormap)
    for i in range(0,width):
        scr.addch(1,i,curses.ACS_HLINE,curses.color_pair(colors[colormap['header-border']]))

def footer(scr,height,width,status,code,colors,colormap):
    for i in range(0,width):
        scr.addch(height-2,i,curses.ACS_HLINE,curses.color_pair(colors[colormap['footer-border']]))
    drawstring(scr,height-1,0,width,status,code,colors,colormap)

def drawlist(scr,list_height,width,offset,songs,item,code,colors,colormap):
    n_songs = len(songs)
    top_half = list_height // 2
    bottom_half = list_height - top_half
    if n_songs <= list_height:
        cl = 0
    elif (item - top_half) < 0:
        cl = 0
    elif (item + bottom_half) > (n_songs - 1):
        cl = n_songs - list_height
    else:
        cl = item - top_half
    for i in range(cl,cl+list_height):
        if i < n_songs:
            song = songs[i]
            if "id" in song:
                s = str(song['track']) + '. ' + song['name']
            else:
                s = song["name"]
            slen = len(s)
            s = s.encode(code)
            try:
                if i == item:
                    drawline(scr,offset+i-cl,width,slen,(s,colors[colormap['current-item']]))
                else:
                    drawline(scr,offset+i-cl,width,slen,(s,colors[colormap['text']]))
            except curses.error:
                pass
        else:
            break

def pause(p,q):
    p.stdin.write(' ')

def play(song):
    with open(os.devnull, 'w') as temp:
        proc =  subprocess.Popen(["mplayer", "-noconfig", "all","%s" % song], stdin=subprocess.PIPE, stdout=temp, stderr=temp)
        return proc

def collect(tree,l,c_item):
    n = 0
    c = 0
    for i in tree:
        if c == c_item:
            n = len(l)
        c += 1
        if 'subtree' in i:
            collect(i['subtree'],l,-1)
        else:
            l.append(i)
    return l, n

def gen_trees(library):
    # Using defaultdict we can group song dicts by artist:
    artists_dict = collections.defaultdict(list)
    for i in library:
        # Add artist of i as key if it doesn't exist and add song as key list item
        artists_dict[i['artist']].append(i)
    # Initialise lists for artists and albums to return
    artists = []
    albums = []
    # With our dictionary of artists we can now loop through and partition in to albums
    for artist,songs_of_artist in artists_dict.iteritems():
        # Create a new default dict for each album in the same way that the artists were partitioned
        albums_of_artist_dict = collections.defaultdict(list)
        for i in songs_of_artist:
            albums_of_artist_dict[i['album']].append(i)
        albums_of_artist = []
        for album,tracks in albums_of_artist_dict.iteritems():
            album_name = album
            if album == "":
                album_name = "Untitled Album"
            albums_of_artist.append({ "name":album_name, 
                                      "subtree":sorted(tracks,key=itemgetter('track')), 
                                      "subtreeline":0 })
        albums = albums + albums_of_artist
        artists.append({ "name":artist, 
                         "subtree":sorted(albums_of_artist,key=lambda x: x['name'].lower()), 
                         "subtreeline":0 })
    return sorted(artists, key=lambda x: x['name'].lower()), sorted(albums, key=lambda x: x['name'].lower())

def list_playlists(api,playlist_ids):
    playlists = []
    for name,pid in playlist_ids['user'].iteritems():
        playlists.append( { "name":name, "subtree":api.get_playlist_songs(pid), "subtreeline":0 } )
    for name,pid in playlist_ids['auto'].iteritems():
        playlists.append( { "name":name, "subtree":api.get_playlist_songs(pid), "subtreeline":0 } )
    return playlists 

def headphones(scr,height,code):
    try:
        with open(os.path.expanduser("~/.thunnerlogo")) as f:
            lines = f.readlines()
            for i in range(0,len(lines)-1):
                if i < height-1:
                    scr.addstr(i,0,lines[i].encode(code))
    except IOError:
        scr.addstr(0,0,"Connecting...".encode(code))

def switch_menu(menu):
    new_list = menu["subtree"]
    new_item = menu["subtreeline"]
    new_n = len(new_list)
    return menu, new_list, new_item, new_n
        
def main():
    
    config = parse_config()

    # Because I'd like to see some umlauts (rather than garbage):
    locale.setlocale(locale.LC_ALL, '')
    code = locale.getpreferredencoding()

    scr = scrinit()
    curses.use_default_colors()
    colors = name_colors(config['colors'])
    colormap = config['assignments']

    api = None
    playing = None
    try:
        height, width = scr.getmaxyx()

        headphones(scr,height,code)

        scr.refresh()

        api = apiinit(config['email'],config['pass'])
        
        if not api.is_authenticated():
            print "Credentials were not accepted. Exiting..."
            return
    
        library = api.get_all_songs()
        artists, albums = gen_trees(library)

        playlist_ids = api.get_all_playlist_ids()
        playlists = list_playlists(api,playlist_ids)

        scr.clear()
        toplevel = { "name":"Toplevel",
                     "subtree":[ {"name":"Artists", "subtree":artists, "subtreeline":0 }, 
                                 {"name":"Albums", "subtree":albums, "subtreeline":0 }, 
                                 {"name":"Songs", "subtree":library, "subtreeline":0 }, 
                                 {"name":"Playlists", "subtree":playlists, "subtreeline":0 }],
                     "subtreeline":0 }
        
        c_menu = toplevel
        c_list = c_menu["subtree"]
        c_item = c_menu["subtreeline"]
        c_n = len(c_list)
        
        previous = []
        
        timeout = 17 # getch timeout in ms

        header_height = 2
        footer_height = 2
        viewheight = height-header_height-footer_height
        
        queue = []
        q_n = 0
        q_length = 0

        status = 'Stopped'
        current_song = None
        
        scr.erase()
        header(scr,width,previous,c_menu,code,colors,colormap)
        footer(scr,height,width,[(status,'status')],code,colors,colormap)
        drawlist(scr,viewheight,width,header_height,c_list,c_item,code,colors,colormap)
        scr.timeout(timeout)
        scr.refresh()
        while True:
            c = scr.getch()
            if c == -1:
                # We've had no input
                # Let's check if a song has stopped playing and we need to play the next in the queue
                if isinstance(playing, subprocess.Popen) and playing.poll() != None:
                    # Song has stopped playing
                    q_n = (q_n + 1) % len(queue)
                    song = queue[q_n]
                    playing = play(api.get_stream_url(song['id']))
                    current_song = song
            elif c == ord('q'): 
                if isinstance(playing, subprocess.Popen):
                    playing.terminate()
                    playing = None
                    break
                else:
                    break
            elif c == ord('\n') or c == ord(']'):
                if c_list:
                    i = c_list[c_item]
                    if isinstance(playing, subprocess.Popen):
                        playing.terminate()
                        queue = []
                    if c == ord('\n'):
                        if c_list != []:
                            l, n = collect(c_list,[],c_item)
                            queue = l
                            q_n = n
                            q_length = len(queue)
                    elif c == ord(']'):
                        if 'id' in i:
                            queue = [i]
                            q_n = 0
                            q_length = 1
                        else:
                            l, n = collect(c_list[c_item]['subtree'],[],'')
                            queue = l
                            q_n = n
                            q_length = len(queue)
                    if queue != [] and q_n < q_length:
                        song = queue[q_n]
                        current_song = song
                        status = 'Playing:'
                        playing = play(api.get_stream_url(song['id']))
            elif c == ord('.'):
                if isinstance(playing, subprocess.Popen):
                    playing.terminate()
                    q_n = (q_n + 1) % len(queue)
                    song = queue[q_n]
                    playing = play(api.get_stream_url(song['id']))
                    current_song = song
                    status = 'Playing:'
            elif c == ord(','):
                if isinstance(playing, subprocess.Popen):
                    playing.terminate()
                    q_n = (q_n - 1) % len(queue)
                    song = queue[q_n]
                    playing = play(api.get_stream_url(song['id']))
                    current_song = song
                    status = 'Playing:'
            elif c == curses.KEY_LEFT:
                if previous:
                    c_menu, c_list, c_item, c_n = switch_menu(previous.pop())
            elif c == curses.KEY_RIGHT:
                if c_list and "subtree" in c_list[c_item]:
                    previous.append(c_menu)
                    c_menu, c_list, c_item, c_n = switch_menu(c_list[c_item])
            elif c == curses.KEY_DOWN: 
                if c_item < c_n-1:
                    c_item += 1
                c_menu["subtreeline"] = c_item
            elif c == curses.KEY_UP:
                if c_item > 0:
                    c_item -= 1
                c_menu["subtreeline"] = c_item
            elif c == ord(' '):
                if status == 'Playing:':
                    status = 'Paused:'
                else:
                    status = 'Playing:'
                q = Queue()
                thread = Thread(target=pause, args=(playing,q))
                thread.daemon = True
                thread.start()
            elif c == curses.KEY_RESIZE:
                height, width = scr.getmaxyx()
                viewheight = height-header_height-footer_height
            # Redraw the screen
            scr.erase()
            header(scr,width,previous,c_menu,code,colors,colormap)
            if current_song == None:
                footer(scr,height,width,[(status,'status')],code,colors,colormap)
            else:
                footer(scr,height,width,[(status,'status'),
                                         (' ','text'),
                                         (current_song['artist'],'current-artist'),
                                         (' - ','footer-sep'),
                                         (current_song['name'],'current-song')],code,colors,colormap)
            drawlist(scr,viewheight,width,header_height,c_list,c_item,code,colors,colormap)
            scr.refresh()
    finally:
        if isinstance(playing, subprocess.Popen):
            playing.terminate()
        scrrelease(scr)
        if api:
            api.logout()
    return

if __name__ == '__main__':
    main()
