# aiolifxc

aiolifxc is a Python 3/asyncio library to control Lifx LED lightbulbs over your LAN.

Most of it was originally taken from
[Meghan Clarkk lifxlan package](https://github.com/mclarkk)
and adapted to Python 3 (and asyncio obviously)

This is a fork from
[François Wautier's package](https://github.com/frawau/aiolifx).
It uses coroutines as opposed to callbacks. If you prefer callbacks,
please see his implementation instead. This was forked from version 0.5.0.

This should be considered Alpha status. While it works, and seems to work
well, there are still changes I wish to make that may break the API.

# Installation

We are on PyPi so

     pip3 install aiolifx
or
     python3 -m pip install aiolifx
     
NOTE: When installing with Python 3.4, the installation produce an error message
      (syntax error). This can be safely ignored. 


# How to use

Essentially, you create an object with at least 2 methods:

    - register
    - unregister

You then start the LifxDiscovery task in asyncio. It will register any new light it finds.
All the method communicating with the bulb can be passed a callback function to react to 
the bulb response. The callback should take 2 parameters:

    - a light object
    - the response message


The easiest way is to look at the file in the examples directory. "Wifi" and "Uptime" use
a callback to print the info when it is returned.

      
In essence, the test program is this

    class bulbs():
    """ A simple class with a register and  unregister methods
    """
        def __init__(self):
            self.bulbs=[]
            
        def register(self,bulb):
            self.bulbs.append(bulb)
            
        def unregister(self,bulb):
            idx=0
            for x in list([ y.mac_addr for y in self.bulbs]):
                if x == bulb.mac_addr:
                    del(self.bulbs[idx])
                    break
                idx+=1
    
    def readin():
    """Reading from stdin and displaying menu"""

        selection = sys.stdin.readline().strip("\n")
        DoSomething()
        
    MyBulbs= bulbs()
    loop = aio.get_event_loop()
    coro = loop.create_datagram_endpoint(
                partial(alix.LifxDiscovery,loop, MyBulbs), local_addr=('0.0.0.0', UDP_BROADCAST_PORT))
    try:
        loop.add_reader(sys.stdin,readin)
        server = loop.create_task(coro)
        loop.run_forever()
    except:
        pass
    finally:
        server.cancel()
        loop.remove_reader(sys.stdin)
        loop.close()
    

Other things worth noting:
    
    -  Whilst LifxDiscover uses UDP broadcast, the bulbs are
       connected with Unicast UDP
       
    - The socket connecting to a bulb is not closed unless the bulb is deemed to have
      gone the way of the Dodo. I've been using that for days with no problem
       
    - You can select to used IPv6 connection to the bulbs by passing an
      IPv6 prefix to LifxDiscover. It's only been tried with /64 prefix.
      If you want to use a /48 prefix, add ":" (colon) at the end of the 
      prefix and pray. (This means 2 colons at the end!)
      
    - I only have Original 1000, so I could not test with other types
      of bulbs
      
    - Unlike in lifxlan, set_waveform takes a dictionary with the right 
      keys instead of all those parameters
