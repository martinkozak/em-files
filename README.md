EventMachine Files
==================

**em-files** solve problem of blocking disk IO when operating with
large files. Use [EventMachine][4] for multiplexing reads and writes
to small blocks performed in standalone EM ticks. They speed down the
file IO operations of sure, but allow running other tasks with them
simultaneously (from EM point of view).

There  is, of sure, question whether this all has sense as `EM::defer` 
is available for handling these blocking tasks. But sometimes are 
situations, in which it's undesirable to execute them in separate thread.

API is similar to classic Ruby file IO represented by [File][1] class.
See an example:
```ruby
require "em-files"
EM::run do
    EM::File::open("some_file.txt", "r") do |io|
        io.read(1024) do |data|     # writing works by very similar
                                    # way, of sure
            puts data
            io.close()
            # it's necessary to do it in block too, because reading
            # is evented
        end
    end
end
```

Support of Ruby API is limited to `#open`, `#close`, `#read` and `#write`
methods only, so for special operations use simply:

```ruby
EM::File::open("some_file.txt", "r") do |io|
    io.native   # returns native Ruby File class object
end
```

### Special Uses

It's possible to use also another IO objects than `File` object by
giving appropriate IO instance instead of filename to methods:

```ruby
require "em-files"
require "stringio"

io = StringIO::new

EM::run do
    EM::File::open(io) do |io|
        # some multiplexed operations
    end
end
```

By this way you can also perform for example more time consuming
operations by simple way (if they can be processed in block manner)
using filters:

```ruby
require "em-files"
require "zlib"

zip = Zlib::Deflate::new
filter = Proc::new { |chunk| zip.deflate(chunk, Zlib::SYNC_FLUSH) }
data = "..."    # some data bigger than big

EM::run do
    EM::File::write(data, filter)   # done in several ticks
end
```

`#write` supports also copying data from another IO stream because it
uses `StringIO` internally. Simply give it IO object instead of
`String`. It will read it until EOF will occur.


Copyright
---------

Copyright &copy; 2011 &ndash; 2015 [Martin Poljak][3]. See `LICENSE.txt` for further details.

[1]: http://www.ruby-doc.org/core/classes/File.html
[2]: http://github.com/martinkozak/em-files/issues
[3]: http://www.martinpoljak.net/
[4]: http://rubyeventmachine.com/
