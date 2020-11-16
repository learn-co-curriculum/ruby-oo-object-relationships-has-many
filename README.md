# Has Many Relationships

## Objectives

- Describe the "has many" relationship between Ruby objects.
- Build classes that produce objects with a "belongs-to" and "has-many"
  relationship.
- Explain why we need to associate objects in this way.

## Introduction

We know that the programs we write are meant to model real-world environments.
This is because the programs we write are designed to carry out real-world jobs
and solve real-world problems. Whether you're creating an app that connects
users around the world in some kind of social network or writing a program for a
major university that manages their course offerings and students, your code
will need to be able to realistically map the relationships between different
entities.

We already know about the "belongs-to" relationship. Let's say we have a `Song`
class that produces individual song objects. Each song belongs to the artist
who wrote it. We can build that relationship by creating an `attr_accessor` in
the `Song` class for `artist`:

```ruby
class Song
  attr_accessor :artist, :name, :genre

  def initialize(name, genre)
    @name = name
    @genre = genre
  end
end
```

If we also have an `Artist` class that looks like this:

```ruby
class Artist
  attr_accessor :name

  def initialize(name)
    @name = name
  end
end
```

We can set the artist attribute of an individual instance of `Song` equal to an
instance of the `Artist` class like this:

```ruby
kiki = Song.new("In My Feelings", "hip-hop")
drake = Artist.new("Drake")

kiki.artist = drake

kiki.artist.name
  # => "Drake"
```

We could just set the `artist` attribute equal to a simple string. However, if
we instead use the `artist=` method to set the attribute equal to a real
instance of the `Artist` class, we are associating our song to a robust object
that has its own attributes and behaviors.

For example, in the code above, we are calling the `#name` method on the artist
associated with `kiki`, a song instance. With method chaining like this, we can
do even more with our code.

## The "has-many" Relationship

The inverse of the "belongs-to" relationship is the "has-many" relationship. If
a song belongs to an artist, then an artist should be able to _have many_ songs.
This makes sense in the real world &mdash; most musical artists have authored
and performed many more than one song.

How can we represent an object's "having many" of something? Well, having many
of something means you own a collection of that thing. Ruby offers us a great
way to store collections of data in list form: arrays.

We would like to be able to call:

```ruby
drake.songs
```

and have returned to us a list, or array, of the songs that Drake has written.
One way to implement this is to create an instance variable, an array, that will
hold the songs belonging to a given artist. As songs are created that belong to
that artist, they can be added to the array.

### Initializing with an Empty Collection

If we want each artist instance to have an instance variable `#songs` to hold
their song instances, it makes sense to create the variable when the artist is
created:

```ruby
class Artist
  attr_accessor :name

  def initialize(name)
    @name = name
    @songs = []
  end
end
```

Here we set an instance variable, `@songs`, equal to an empty array. Recall that
we use instance variables to store the attributes of a given instance of a
class. This instance variable is set equal to an empty array because our artist
doesn't have any songs yet.

Let's write the method that will allow us to add some.

### Adding items to the collection

Whose responsibility is it to add a new song to a given artist's collection?
Well, at what point in time does an artist add another song to their repertoire?
When they write a new song. It makes sense, therefore, that the Artist class be
responsible for adding the new song to the artist's collection at the time the
song is created.

Let's write the method that adds songs to an artist's collection in the `Artist`
class:

```ruby
class Artist
  attr_accessor :name

  def initialize(name)
    @name = name
    @songs = []
  end

  def add_song(song)
    @songs << song
  end
end
```

Now we can execute the following code:

```ruby
drake = Artist.new("Drake")
drake.add_song("In My Feelings")
drake.add_song("Hotline Bling")
```

Next we need a method that will allow a given artist to show us all of the songs
in their collection. Let's do it.

### Exposing the Collection

Let's write an instance method, `#songs`, that we can call on an individual
artist to return the list of songs that the artist has.

```ruby
class Artist
  attr_accessor :name

  def initialize(name)
    @name = name
    @songs = []
  end

  def add_song(song)
    @songs << song
  end

  def songs
    @songs
  end
end
```

The `#songs` method simply returns the `@songs` array, which contains the list of
songs that the artist has many of.

Let's try it out:

```ruby
drake.songs
  # => ["In My Feelings", "Hotline Bling"]
```

### Relating Objects with "belongs to" and "has many"

"Wow, those look like interesting songs," you might be thinking. "I wonder what
kind of music Drake makes." Well, let's ask `drake` to tell us the genres of the
songs he has many of.

Oh no! We can't do that because `drake`'s songs are simply a list of strings. We
can't ask a plain old string what genre it has &mdash; it will have no idea what
we are talking about.

This is the limitation of one-sided relationships. Earlier we discussed the
drawbacks of associating a given song to a string that contains an artist's name
instead of to a real `Artist` instance. The same drawbacks apply to associating
a given artist to a list of strings: with this setup, we are limited to
references to a given artist's songs by their name alone. We cannot associate
any further information to an artist's songs or enact any further behavior on an
artist's songs.

Let's fix this now. Instead of calling the `#add_song`  method with an argument
of a string, let's call that method with an argument of a real song object:

```ruby
kiki = Song.new("In My Feelings", "hip-hop")
hotline = Song.new("Hotline Bling", "pop")

drake.add_song(kiki)
drake.add_song(hotline)

drake.songs
  # =>[#<Song:0x007fa96a878348 @name="In My Feelings", @genre="hip-hop">, #<Song:0x007fa96a122580 @name="Hotline Bling", @genre="pop">]
```

Great, now our artist has many songs that are real, tangible `Song` instances,
not just strings.

We can do several useful things with this collection of real song objects, such
as iterate over them and collect their genres:

```ruby
drake.songs.collect do |song|
  song.genre
end
  # => ["hip-hop", "pop"]
```

#### Object Reciprocity

Now that we can ask our given artist for their songs, let's make sure that we can
ask an individual song for its artist:

```ruby
kiki.artist
  # => nil
```

Although we do have an `attr_accessor` for `artist` in our `Song` class, this
particular song doesn't seem to know that it belongs to Drake. That is because
our `#add_song` method only accomplished associating the song object to the
artist object. Our artist knows it has a collection of songs and knows how to
add songs to that collection. But, we didn't tell the song we added to the
artist that it belonged to that artist.

![belongs to](https://curriculum-content.s3.amazonaws.com/module-1/ruby-oo-relationships/has-many/Image_138_CodeObjectsConvo%28A%29.png)

Let's fix that now. Telling a song that it belongs to an artist should happen
when that song is added to the artist's `@songs` collection. Consequently, we
will write the code that accomplishes this inside our `#add_song` method:

```ruby
class Artist
  attr_accessor :name

  def initialize(name)
    @name = name
    @songs = []
  end

  def add_song(song)
    @songs << song
    song.artist = self
  end

  def songs
    @songs
  end
end
```

Let's take a closer look at the code in our `#add_song` method:

```ruby
def add_song(song)
  @songs << song
  song.artist = self
end
```

Here, we use the `self` keyword to refer to the artist on which we are calling
this method. We call the `#artist=` method on the song that is being passed in
as an argument and set it equal to `self` &mdash; the artist.

Let's try calling `#add_song` again:

```ruby
drake.add_song(kiki)
```

Now, we should be able to ask `kiki` for its artist:

```ruby
kiki.artist.name
  # => "Drake"
```

We did it! Not only does an artist have many songs, but a song belongs to an
artist and we built a method that enacts those associations at the appropriate
time.

## Maintaining a Single Source of Truth

The `#add_song` method works, but there is a flaw in this setup.

```ruby
def add_song(song)
  @songs << song
  song.artist = self
end
```

With this implementation, we're maintaining this relationship on both the `Song`
instance and the `Artist` instance. We've done this so that an artist knows
which songs it has, and a song knows the artist it belongs to. However, keeping
this information maintained on both sides of the relationship means there are
**_two sources of truth_**: the song's knowledge of who its artist is
(established by using the `artist=` method) and the artist's knowledge of which
songs it has (the list of songs in the artist's `@songs` array).

So why is this a problem? Well, what happens if we _don't_ consistently use the
`add_song` method? What if, instead, somewhere along the lines we did something
like this:

```ruby
lil_nas_x = Artist.new("Lil Nas X")
old_town_road = Song.new("Old Town Road","hip-hop")

old_town_road.artist = lil_nas_x

old_town_road.artist.name #=> "Lil Nas X"
lil_nas_x.songs #=> []
```

Now, the `Song` instance `old_town_road` is associated with an artist, but
`lil_nas_x` **_does not know about_** `old_town_road` because the song was not
added to `lil_nas_x`'s `@songs` array. We have two sources of truth about
artists and their songs &mdash; one from the artist side and one from the song
side &mdash; and they're not aligned.

A better way to approach this would be to figure out how to maintain our
"has-many" / "belongs-to" relationship _on only one side of the relationship_.
We can do that by having the `Song` class keep a list of _all of the songs by
all artists_, and writing the `#songs` method in our Artist class to query that
list, asking for the songs that belong to a given artist.

![belongs to](https://curriculum-content.s3.amazonaws.com/module-1/ruby-oo-relationships/has-many/Image_138_CodeObjectsConvo%28B%29.png)

This may become clearer if we make some updates to `Song` and `Artist`. Say, for
instance, in `Song`, we set up a class variable, `@@all`, set to an empty Array,
and a getter method, `.all`. This way, when a song is initialized, we can push
the instance into the `@@all` array and then use `Song.all` to retrieve all
`Song` instances:

```ruby
class Song
  attr_accessor :artist, :name, :genre

  @@all = []

  def initialize(name, genre)
    @name = name
    @genre = genre
    save
  end

  def save
    @@all << self
  end

  def self.all
    @@all
  end
end
```

Okay, so now that we can get all songs, we should be able to do things like
this:

```ruby
lil_nas_x = Artist.new("Lil Nas X")
rick = Artist.new("Rick Astley")

old_town_road = Song.new("Old Town Road","hip-hop")
never_gonna_give_you_up = Song.new("Never Gonna Give You Up","pop")

old_town_road.artist = lil_nas_x
never_gonna_give_you_up.artist = rick

Song.all.first.name #=> "Old Town Road"
Song.all.first.genre #=> "hip-hop"
Song.all.first.artist #=> #<Artist:0x00007ff1d90dbf38 @name="Lil Nas X", @songs=[]>
Song.all.first.artist.name #=> "Lil Nas X"


Song.all.last.name #=> "Never Gonna Give You Up"
Song.all.last.genre #=> "pop"
Song.all.last.artist #=> #<Artist:0x00007ff1d90dbf38 @name="Rick Astley", @songs=[]>
Song.all.last.artist.name #=> "Rick Astley"
```

Now we've got a way to get all songs, so if we want a list of all the songs that
belong to a particular artist we can just _select_ the appropriate songs:

```ruby
Song.all.select {|song| song.artist == lil_nas_x}
#=> [#<Song:0x00007ff1da1d3228 @name="Old Town Road", @genre="hip-hop", @artist=#<Artist:0x00007ff1d90dbf38 @name="Lil Nas X", @songs=[]>>]

Song.all.select {|song| song.artist == rick}
#=> [#<Song:0x00007ff1da87bc38 @name="Never Gonna Give You Up", @genre="pop", @artist=#<Artist:0x00007ff1da20b150 @name="Rick Astley", @songs=[]>>]
```

So we use `.select` to iterate through `Song.all` and return the subset of songs
associated with a particular artist. We can incorporate this directly into our
`Artist` class, replacing the implementation of the `#songs` method so that it
_selects_ instead of returning the `@songs` instance variable. Because `#songs`
is an instance method, we can use `self` to represent the `Artist` instance this
method is called on:

```ruby
class Artist
  ...

  def songs
    Song.all.select {|song| song.artist == self}
  end
end
```

Now that we can get the necessary information by selecting from `Song.all`, we
no longer need the `@songs` instance variable in our `Artist` class. We can get
rid of that, and update `#add_song` accordingly:

```ruby
class Artist
  attr_accessor :name

  def initialize(name)
    @name = name
  end

  def add_song(song)
    song.artist = self
  end

  def songs
    Song.all.select {|song| song.artist == self}
  end
end
```

With this implementation, we're able to achieve a "has-many" / "belongs-to"
relationship while maintaining a single source of truth! Not only that, we were
able to simplify the `Artist` class without losing any functionality!

Now, let's go back to the original example in this section. With our new setup,
the issue of maintaining both sides of the relationship is solved. Simply by
telling the song instance which artist it belongs to, we are able to access the
list of songs that belong to a given artist. This works whether we use the
Song class's `artist=` method or the Artist class's `add_song` method:

Calling the `artist=` method on a Song instance:

```ruby
lil_nas_x = Artist.new("Lil Nas X")
old_town_road = Song.new("Old Town Road","hip-hop")

old_town_road.artist = lil_nas_x

old_town_road.artist.name #=> "Lil Nas X"
lil_nas_x.songs #=> [#<Song:0x00007fb46b0a1c08 @name="Old Town Road", @genre="hip-hop", @artist=#<Artist:0x00007fb46b0e3748 @name="Lil Nas X">>]
```

Calling the `add_song` method on an Artist instance:

```ruby
rick = Artist.new("Rick Astley")
never_gonna_give_you_up = Song.new("Never Gonna Give You Up","pop")
rick.add_song(never_gonna_give_you_up)

rick.songs #=> [#<Song:0x00007fb46b0b97b8 @name="Never Gonna Give You Up", @genre="pop", @artist=#<Artist:0x00007fb46a903000 @name="Rick Astley">>]
never_gonna_give_you_up.artist #=> #<Artist:0x00007fb46a903000 @name="Rick Astley">
```

## Extending the Association and Cleaning up our Code

The code we have so far is pretty good. The best thing about it though is that
it accommodates future change. We've built solid associations between our
`Artist` and `Song` classes via our has many/belongs to code. With this
foundation we can make our code even better in the following ways:

### The `#add_song_by_name` Method

As it currently stands, we have to *first* create a song and *then* add it to a
given artist's collection of songs. We are lazy programmers, if we could combine
these two steps, that would make us happy. Furthermore, if you think about our
domain model, i.e. the program we are writing to model the real-world
environment of an artist and their songs, the current need to create a song and
then add it to an artist doesn't really make sense. A song doesn't exist
*before* an artist creates it.

Instead, let's build a method `#add_song_by_name`, that takes in an argument of
a name and genre and both creates the new song *and* adds that song to the
artist's collection.

```ruby
class Artist
  ...

  def add_song_by_name(name, genre)
    song = Song.new(name, genre)
    song.artist = self
  end
```

Here we tell the song that it belongs to the artist, just as we do in our
`#add_song` method, and we also create a new song instance using the name and
genre from the arguments.

![belongs to](https://curriculum-content.s3.amazonaws.com/module-1/ruby-oo-relationships/has-many/Image_138_CodeObjectsConvo%28C%29.png)

This is not only neater and more elegant &mdash; we no longer have to create a
new song on a separate line *every time* we want to add one to an artist &mdash;
but it makes more sense.

### The `#artist_name` Method

Since we've already set up these great associations between instances of the
`Song` and `Artist` class, we can use them to build other helpful methods.

Currently, to access the name of a given song's artist, we have to chain our
methods like this:

```ruby
kiki.artist.name
  # => "Drake"
```

We can imagine knowing the name of an artist that a particular song belongs to
would be helpful and probably used in mulitple situations. Rather than having to
chain multiple methods, wouldn't it be nice if we have one simple and
descriptive method that could return the name of a given song's artist? Let's
build one!

```ruby
class Song
  ...

  def artist_name
    self.artist.name
  end
```

Now we can call:

```ruby
kiki.artist_name
  # => "Drake"
```

Much better. Notice that we used the `self` keyword inside the `#artist_name`
method to refer to the instance of `Song` on which the method is being called.
Then we call `#artist` on that song instance. This returns the `Artist`
instance associated with the song. Chaining a call to `#name` after that is
equivalent to saying: call `#name` on the return value of `self.artist`, i.e.
call `#name` on the artist of this song.

## Conclusion

Using the foundation of "has-many" / "belongs-to" associations, we can create
many different useful methods. We can write methods like `add_song_by_name`
that handle initializing and associating instances. We can also create methods
like `artist_name` that can simplify retrieving information from an associated
instance.

Establishing both "has-many" and "belongs-to" associations between two objects
allows us to ask a song who its artist is, and ask an artist what their songs
are. We've established a bi-directional relationship! Can you think of any other
real world relationships where these associations could be applied?
