rake-gcc
========

A DSL for using gcc from rake.

Usage
=====

Here is an example rage-gcc file:

```ruby

build_target :debug do
  compiler "g++"
  
  before do
    sh "windres \"-I#{$WXWIDGETS}/include\" resources.rc #{@build_target.name}/obj/resources.o"
  end
  
  compile do
    define :DEBUG
    
    flag "-std=c++11"
    
    search [
      "include",
      "/some/other/library/include"
    ]
    
    sources "src/**/*.{c,cpp}"
  end
  
  link do
    search "/some/other/library/lib",
      "../and/yet/another/lib"
    
    object "#{@build_target.name}/obj/resources.o"
    
    libs "otherlibrary", "yetanother"
    
    artifact "app.exe"
  end
  
  copy "app.xrc"
end

build_target :release, :debug do
  compile do
    undefine :DEBUG
    define :RELEASE
  end
end
```

Here it is again, fully explained:

```ruby

# `build_target` creates a new... build target.
# - All tasks defined for this target are namespaced under the given name,
#   in this case "debug"
# - Defined tasks include TARGET:build & TARGET:clean,
#   in this case that's `debug:build` & `debug:clean`
build_target :debug do
  # `compiler` allows you to set the compiler to use (gcc vs g++)
  compiler "g++"
  
  # Your `before` block will be run after the directories are set up for your 
  # build, but before any other tasks are run. Note that you can access the
  # BuildTarget object by @build_target. This way you can refer to the name
  # indirectly, allowing this before block to work correctly for any build targets
  # that inherit from this one.
  before do
    sh "windres \"-I#{$WXWIDGETS}/include\" resources.rc #{@build_target.name}/obj/resources.o"
  end
  
  # You specify compilation options in the `compile` block...
  compile do
    # ...like defines (as in -DDEBUG)
    define :DEBUG
    
    # You can also specify flags
    flag "-std=c++11"
    
    # Or where to search for included files (-Iinclude -I/some/other/library/include)
    # Note that all of the compile/link dsl methods accept single arguments, arrays, 
    #or multiple arguments.
    search [
      "include",
      "/some/other/library/include"
    ]
    
    # And of course, you can specify what your source files are. 
    # (This takes anything you can give to a Rake::FileList)
    sources "src/**/*.{c,cpp}"
  end
  
  # You specify linker options in the `link` block...
  link do
    # ...like where to search for library files. (-L/some/other/library/lib ...)
    search "/some/other/library/lib",
      "../and/yet/another/lib"
    
    # rake-gcc knows about the object files it creates for your compiled
    # sources, but you might like to tell it about some others to include, 
    # like your resources.o file.
    object "#{@build_target.name}/obj/resources.o"
    
    # You can also tell it what libs to link against.
    # Note that if you supply an array, or multiple arguments, they will
    # be listed in the provided order on the command line. If, however,
    # you call `lib` or `libs` again, the provided arguments are prepended
    # to the library list, and so will appear first on the command line.
    # This may seem odd at first, but it's useful when inheriting from another
    # build target that has already defined libraries that you depend on.
    libs "otherlibrary", "yetanother"
    
    # Finally, you can specify the name of the output artifact
    artifact "app.exe"
  end
  
  # `copy` tells rake-gcc to copy the supplied files into the target folder.
  # - Like most other DSL methods here, you can supply multiple arguments or arrays.
  # - Like `sources`, this method also takes anything you can give to a Rake::FileList
  # - If an argument in a hash like {some_file_list => other_file_list}, rake-gcc
  #   will copy each file from some_file_list to the target directory, changing the
  #   relative paths from there to match other_file_list. See wx-ruby-template for
  #   an example.
  copy "app.xrc"
end

# You can inherit another configuration by providing its name as the
# second argument to `build_target`
build_target :release, :debug do
  # Then you can alter the configuration only as needed,
  # without having to duplicate the common tasks and settings.
  # Here, we're changing the list of defines for the compiler.
  compile do
    undefine :DEBUG
    define :RELEASE
  end
end
```

To see all of the methods available in the DSL, checkout all the classes ending in "DslContext" in the library.

For a full working example, check out the [Rakefile](https://github.com/jbreeden/wx-ruby-template/blob/master/Rakefile.rb) for [wx-ruby-template](https://github.com/jbreeden/wx-ruby-template)

TODO
====

- Add `desc` blocks so tasks show up with `rake -T`
- Split classes/module into separate files
- Gemify

LICENSE
-------

(The MIT License)

Copyright (c) 2014 Jared Breeden

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
