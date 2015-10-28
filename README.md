# Ruby Objects Has Many

## Outline

Song belongs to artist but how do we represent artists having songs

Songs property
Artist.songs = []
Artists.songs
Artist.songs <<
Artist.new, artist.songs <<
Initialize with array

Song.artist = artist, artist.songs
Object reciprocity
Overriding artist =
artist.add_song(song) for reciprocity
stopping infinite callback loop

complete domain model of has_many/belongs_to

song.artist_name = "" extension
artist.add_song_by_name("") extension

stretch: what to do about people using artist.songs << and not artist.add_song?
