    8888888888P        d8888 8888888b.  8888888b.     d8888          @@@    @@@ 
          d88P        d88888 888   Y88b 888   Y88b   d88888        @@@@@@@@@@@@@@
         d88P        d88P888 888    888 888    888  d88P888       @@@@@@@@@@@@@@@@
        d88P        d88P 888 888   d88P 888   d88P d88P 888      @@@@@@@@  @@@@@@@@
       d88P        d88P  888 8888888P"  8888888P" d88P  888      @@@            @@@
      d88P        d88P   888 888        888      d88P   888      @@   @@@@@@@@   @@
     d88P        d8888888888 888        888     d8888888888           @@@@@@@@    
    d8888888888 d88P     888 888        888    d88P     888            @@@@@@     

# Node development for the lazy

If you can describe it in 495 characters, why on earth should it take 879?

Zappa is a [CoffeeScript](http://coffeescript.org)-optimized interface to [Express](http://expressjs.com) and [Socket.IO](http://socket.io) that makes this:

```coffee
require('zappa') ->
  Gizmo = require './model/gizmo'
  
  @use 'bodyParser', 'methodOverride', @app.router, 'static'

  @configure
    development: => @use errorHandler: {dumpExceptions: on}
    production: => @use 'errorHandler'

  @get '/': -> @render 'index'
  
  @get '/gizmos/:id': ->
    Gizmo.findById @params.id, (err, gizmo) =>
      @render index: {err, gizmo}

  @on connection: ->
    @emit welcome: {time: new Date()}

  @on shout: ->
    @broadcast shout: {@id, text: @data.text}
```

Equivalent to this:

```coffee
express = require 'express'
app = express.createServer()
io = require('socket.io').listen(app)

Gizmo = require './model/gizmo'

app.use express.bodyParser()
app.use express.methodOverride()
app.use app.router
app.use express.static __dirname + '/public'

app.configure 'development', ->
  app.use express.errorHandler dumpExceptions: on

app.configure 'production', ->
  app.use express.errorHandler()

app.get '/', (req, res) -> res.render 'index'

app.get '/gizmos/:id', (req, res) ->
  Gizmo.findById req.params.id, (err, gizmo) ->
    res.render 'index', {err, gizmo}

io.sockets.on 'connection', (socket) ->
  socket.emit 'welcome', time: new Date()

  socket.on 'shout', (data) ->
    socket.broadcast.emit 'shout',
      id: socket.id, text: data.text

app.listen 3000

console.log "Express server listening on port %d in %s mode",
  app.address().port, app.settings.env
```

And throws in some additional features while at it:

```coffee
require('zappa') ->
  @enable 'default layout', 'serve jquery',
    'serve sammy', 'minify'

  @get '/': ->
    @render 'index'

  @on connection: ->
    @emit welcome: {result: sum 1, 2}
  
  @shared '/shared.js': ->
    root = window ? global
    root.sum = (x, y) -> x + y
  
  @client '/index.js': ->
    @connect()

    @get '#/route': ->
      $('body').append 'client routes!'

    @on welcome: ->
      $('body').append "welcomed: #{sum @data.result, 2}"
  
  @view index: ->
    @title = 'PicoChat!'
    @scripts = ['/socket.io/socket.io', '/zappa/jquery',
      '/zappa/sammy', '/zappa/zappa', '/shared', '/index']
  
    h1 @title
```

## Learn More

- Get the gist with the [crash course](http://zappajs.org/docs/crashcourse)

- Check the [API reference](http://zappajs.org/docs/0.3-gumbo/reference)

- See the [examples](https://github.com/mauricemach/zappa/tree/master/examples) included with the source

- Read the [annotated source](http://zappajs.org/docs/zappa.html) generated by [docco](http://jashkenas.github.com/docco/)

## Other resources

- The source code [repository](http://github.com/mauricemach/zappa) at github

- Questions, suggestions? Drop us a line on the [mailing list](http://groups.google.com/group/zappajs)

- Rather do it realtime? Join the IRC channel on freenode: [#zappajs]((irc://irc.freenode.net/zappajs)

- Found a bug? Open an [issue](http://github.com/mauricemach/zappa/issues) at github

- Check the project's history at the [change log](https://github.com/mauricemach/zappa/blob/master/CHANGELOG.md)

- Migrating from an earlier version? Read the announcements ([0.2.x](http://zappajs.org/docs/0.2-peaches/announcement)/[0.3.x](http://zappajs.org/docs/0.3-gumbo/announcement)) for an overview on changes, and follow the TL;DR migration guides ([0.2.x](http://zappajs.org/docs/0.2-peaches/migration)/[0.3.x](http://zappajs.org/docs/0.3-gumbo/migration))

- Deploying to heroku? Check [this blog post](http://blog.superbigtree.com/blog/2011/08/19/hosting-zappa-0-2-x-on-heroku/)

## Thanks loads

To all people behind the excellent libs that made this little project possible, more specifically: 

- Jeremy Ashkenas for CoffeeScript, the "little" language is nothing short of revolutionary to me.

- TJ Holowaychuk for the robust and flexible cornerstone that's Express.

- Guillermo Rauch for solving (as far as I'm concerned) the comet problem once and for all.

- Ryan Dahl for Node.js, without which nothing of this would be possible.

Also:

- Blake Mizerany for Sinatra, the framework that made me redefine simple.

- why the lucky stiff, for making me redefine hacking.

- And last but not least Frank Zappa, for the spirit of nonconformity and experimentation that inspires me to push forward. Not to mention providing the soundtrack.

"Why do you necessarily have to be wrong just because a few million people think you are?" - FZ