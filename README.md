# Ruby Objects: The "Has Many" Relationship

## Objectives

1. Understand the concept of the "has many" relationship between Ruby objects. 
2. Learn how to build class that produce objects with a belongs to and has many relationship. 
3. Understand why we need to associate objects in this way. 

## Introduction

We know that the programs we write are meant to model real-world environments. This is because the programs we write are designed to carry out real-world jobs and solve real-world problems. Whether you're creating an app that connects users around the world in some kind of social network or writing a program for a major university that manages their course offerings and students, your code will need to be able to realistically map the relationships between different entities. 

We already know about the "belongs to" relationship. Let's say we have a `Song` class that produces individual song objects. Each song belongs to the artist that wrote it. We can build that relationship by creating an `attr_accessor` in the `Song` class for `artist`:

```ruby
class Song
	attr_accessor :artist, :name
	
	def initialize(name)
		@name = name
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

We can set an individual instance of `Song` equal to an instance of the `Artist` class like this:

```ruby
99_problems = Song.new("99 Problems")
jay_z = Artist.new("Jay-Z")

99_problems.artist = jay_z

99_problems.artist.name
  # => "Jay-Z"
```

The benefit here is that in setting the `artist=()` method equal to a real instance of the `Artist` class, instead of equal to a simple string, we are associating our song to a robust object that has its own attributes and behaviors. 

For example, in the code above, we are calling the `#name` method on the artist of `99_problems`. With method chaining like this, we can do even more with our code. 

The inverse of the "belongs to" relationship is the "has many" relationship. If a song belongs to an artist, then an artist should be able to have many songs. This makes sense in the real-world––most musical artists have authored and performed many more than one song. 

Let's take a closer look. 

## The "has many" Relationship

How can we represent an object's "having many" of something? Well, having many of something means you own a collection of that thing. Ruby offers us a great way to store collections of data in list form: arrays. 

We would like to be able to call:

```ruby
jay_z.songs
```

And have returned to us a list, or array, of the songs that Jay-Z has written. A given artist should start, or be initialized, with a songs collection that is empty. Later, we will write a method that adds songs to that collection. 

### Initializing with an empty collection

```ruby
class Artist

	attr_accessor :name

	def initialize(name)
	  @name = name
	  @songs = []
	end
end
```

Here we set an instance variable, `@songs`, equal to an empty array. Recall that we use instance variables to store the attributes of a given instance of a class. This instance variable is set equal to an empty array because our artist doesn't have an songs yet. 

Let's write the method that will allow us to add some. 

### Adding items to the collection

Whose responsibility is it to add a new song to a given artist's collection? Well, at what point in time does an artist add another song to his or her repetoir? When that artist writes a new song. Consequently, it isn't the song's responsibility to add itself to the artist's collection of songs, it is the artist's responsibility to add a new song to their collection. 

That's why we'll write the method that adds songs to an artist's collection in the `Artist` class:

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
jay_z = Artist.new("Jay-Z")
jay_z.add_song("99 Problems")
jay_z.add_song("Crazy in Love")
```

Now we need a method that will allow a given artist to show us all of the songs in their collection. Let's do it. 

### Exposing the Collection

Let's write an instance method, `#songs`, that we can call on an individual artist to return the list of songs that artist has. 

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

The `#songs` method simply return the `@songs` array, which contains the list of songs that the artist has many of. 

Let's try it out:

```ruby
jay_z.songs
  # => ["99 Problems", "Crazy in Love"]
```

### Relating Objects with "belongs to" and "has many"

Wow, those look like interesting songs, you might be thinking. I wonder what kind of music Jay-Z makes, you might be wondering if you live on Mars. Well, let's ask `jay_z` to tell us the genres of the songs he has many of. 

Oh no! We can't do that because `jay_z`'s songs are simply a list of strings. We can't ask a plain old string what genre it has, it will have no idea what we are talking about. 

This is the limitation of one-sided relationships. Just like associating a given song to a string that contains an artist's name instead of to a real `Artist` instance had it's drawbacks, so to does associated a given artist to a list of strings. With this set up, we are limited to refers to a given artist's songs by their name alone. We cannot associate any further information to an artist's songs or enact any further behavior on an artist's songs. 

Let's fix this now. Instead of calling the `#add_song`  method with an argument of a string, let's call that method with an argument of a real song object:

```ruby
99_problems = Song.new("99 Problems", "rap")
crazy_in_love = Song.new("Crazy in Love", "pop")

jay_z.add_song(99_problems)
jay_z.add_song(crazy_in_love)

jay_z.songs
  # =>[#<Song:0x007fa96a878348 @name="99 Problems", @genre="rap">, #<Song:0x007fa96a122580 @name="Crazy in Love", @genre="pop">]
```

Great, now our artist has many songs that are real, tangible `Song` instances, not just strings. 

We can do a number of useful things with this collection of real song objects, such as iterate over them and collect their genres:

```ruby
jay_z.songs.collect do |song|
	song.genre
end
  # => ["rap", "pop"]
```

#### Object Reciprocity

Now that we can ask our given artist for his songs, let's make sure that we can ask an individual song for it's artist:

```ruby
crazy_in_love.artist
  # => nil
```

Although we do have an `attr_accessor` for `artist` in our `Song` class, this particular song doesn't seem to know that it belongs to Jay-Z. That is because our `#add_song` method only accomplished associating the song object to the artist object. Our artist knows it has a collection of songs and knows how to add songs to that collection. But, we didn't tell the song that we added to the artist that it belonged to that artist. 

Let's fix that now. Telling a song that it belongs to an artist should happen when that song is added to the artist's `@songs` collection. Consequently, we will write the code the accomplishes this inside our `#add_song` method:

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
		@@songs 
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

Here, we use the `self` keyword to refer to the artist on which we are calling this method. We call the `#artist=()` method on the song that is being passed in as an argument and set it equal to `self`––the artist. 

Let's try calling `#add_song` again:

```ruby
jay_z.add_song(crazy_in_love)
```

Now, we should be able to ask `crazy_in_love` for it's artist:

```ruby
crazy_in_love.artist.name
  # => "Jay-Z"
```

We did it! Not only does an artist have many songs, but a song belongs to an artist and we built a method that enacts those associations at the appropriate time. 

## Extending the Association and Cleaning up our Code

The code we have so far is pretty good. The best thing about it though, is that it accommodates future change. We've build solid associations between our `Artist` and `Song` class via our has many/belongs to code. With this foundation we can make our code even better in the following ways:

### The `#add_song_by_name` Method

As it currently stands, we have to *first* create a song and *then* add it to a given artists collection of songs. We are lazy programmers, if we could combine these two steps, that would make us happy. Furthermore, if you think about our domain model, i.e. the program we are writing to model the real-world environment of an artist and his or her songs, the current need to create a song and then add it to an artist doesn't really make sense. A song doesn't exist *before* an artist creates it. 

Instead, let's build a method `#add_song_by_name`, that takes in an argument of a name and genre and both creates the new song *and* adds that song to the artist's collection. 

```ruby
class Artist 
  ...
  
  def add_song_by_name(name, genre)
    song = Song.new(name, genre)
    @songs << song
    song.artist = self
  end
```

Here we use the logic of our original `#add_song` method, which adds a song to an artist's collection and tells that song that it belongs to that particular artist. But, we also create a new song using the name and genre from the arguments. 

This is not only neater and more elegant––now we don't have to create a new song on a separate line *every time* we want to add one to an artist––but it makes more sense. 

### The `#artist_name` Method

Since we've already set up these great associations between instances of the `Song` and `Artist` class, we can use them to build other helpful methods. 

Currently, to access the name of a given song's artist, we have to chain our methods like this:

```ruby
crazy_in_love.artist.name
  # => "Jay-Z"
```

That's not very elegant. Wouldn't it be nice if we have one simple and descriptive method that could return the name of a given song's artist? Let's build one!

```ruby
class Song
  ...
  
  def artist_name
    self.artist.name
  end
```

Now we can call:

```ruby
crazy_in_love.artist_name
  # => "Jay-Z"
```

Much better. Notice that we used the `self` keyword inside the `#artist_name` method to refer to the instance of `Song` on which the method is being called. Then we call `#artist` on that song instance. This would return the `Artist` instance associated to the song. Chaining a call to `#name` after that is equivalent to saying: call `#name` on the return value of `self.artist`, i.e. call `#name` on the artist of this song. 

These are only a few of the ways in which you can extend, or build on, the foundational has many and belongs to associations. 

<a href='https://learn.co/lessons/ruby-objects-has-many-readme' data-visibility='hidden'>View this lesson on Learn.co</a>
