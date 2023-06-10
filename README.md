# Opinionated Commonplace Book and Incremental Notes

## A fool takes notes

There is [a lot of note-taking software out there](https://en.wikipedia.org/wiki/Comparison_of_note-taking_software), and the trouble with most of it is that it's too darn complicated. Which is a shame, because note-taking is something a lot of people do with computers.

There are exceptions like [Google Keep](https://keep.google.com/#home) and [thinktype](https://thinktype.app/) but it seems to me most of the big names are useless and confusing to most people, including me, because they require so much investment in learning how to use them. You need something simple and casual to take notes (like, say, a pen and paper) not something complicated.

## Types of note taking

So, take a breath and look at the landscape. There are, amongst other approaches:

1. [Commonplace books](https://en.wikipedia.org/wiki/Commonplace_book) Leonardo da Vinci kept all of his notes in one big book. If he liked something he put it down. This is known as a commonplace book, and it is about how detailed your note-taking system should be unless you plan on thinking more elaborately than Leonardo da Vinci.
2. [Zettelkasten](https://en.wikipedia.org/wiki/Zettelkasten) or a card file: small items of information stored on paper slips or cards that may be linked to each other through subject headings or other metadata such as numbers and tags.
3. [Incremental notes](https://thesephist.com/posts/inc/) are like a diary, but for notes: start a new page every day and fill it with what you're doing, not doing, or reading, or whatever.
4. Spreadsheets. Some people know a bit about spreadsheets and fill them will all kinds of stuff, like the [list of clothes they own and their washing schedule](https://old.reddit.com/r/AskUK/comments/145vqof/whats_a_weird_thing_you_do_to_organise_yourself/). People use the tool they know, rather than the tool that would do the job best - it's like they are pounding in screws with a hammer or opening a paint tin with a the point of a kitchen knife, and it upsets me. We will talk no more about trying to model a gloriously messy world with rectangular grids of values.
5. Databases. Well, yes, but then everything's a database of sorts, isn't it? If by 'database' you mean 'relational database', then we're back to point 4, which we don't talk about.

The aim here is to make something simple that explicitly covers commonplace books and incremental notes, and enable a little of the functionality of zettelkasten using hashtags. After playing around with some designs for a while, I thought it might work to split commonplace books and incremental notes into two similar apps, so that they can be run side-by-side, so text can be cut and pasted between them.

## A little app called `com`

`com` is a simple app that allows you to write and retrieve commonplace book notes. You can have more than one commonplace book; the default one is called `Default`, but you can create others. (You could, for instance, create separate books for 'Cooking', 'Hamster Collecting' and 'Work', but I'm not sure I'd recommend it, because lines can get blurry. Just ask Leonardo da Vinci.)

![screenshot](https://github.com/oddstream/nincomp/blob/60129c47db7d5926ad494b3f07d4f9d84d6ef988/screenshots/com.png)

The user interface only has four elements:

1. The text of the current note
2. An entry box where you can type for a word that exists in a note (in the current book)
3. A list of note titles that have been found by typing into the entry box above it
4. A toolbar, containing commands to: change the current book or create a new book; create a new note; search for hashtags in the book.

## A little app called `inc`

`inc` is a simple app that allows you to write and retrieve incremental notes. You can have more than one notebook; the default one is called `Default`, but you can create others.

![screenshot](https://github.com/oddstream/nincomp/blob/d6909fcd888d94aadec1d619def96643d1abe500/screenshots/inc.png)

`inc` generates a new note for you everyday (but you can still edit old notes, or create notes in the future). There is no explicit 'create note' feature, everyday has it's own note.

I toyed with the idea that notes from days before today cannot be edited. Think of it like this: last October, your favorite color was red, so you made a note of it. Now, your favorite color is blue. So, should you go back and edit the note from October, removing your choice from history, or just make a new note? I think the user can just resolve not to edit old notes, rather than have the app decide that for them.

The idea came from [The Sephist's article](https://thesephist.com/posts/inc/) and from using [rednotebook](https://rednotebook.app) for a while.

## Workflow

The general idea is have an instance of `inc` open, where you put notes and bits of text as they come up during the day. (You can have more than one instance of `inc` open, one for each book, if you use multiple books.) Then, have one or more instances of `com` open, and copy-and-paste text from `inc` to `com` as that information endures or needs categorization.

Thereafter, because all the notes are just text files in directory trees, they can be manipulated, exported, reformatted by worthier and more appropriate tools.

## Implementation

`com` and `inc` are written in [Go](https://go.dev/), with the user interface done using the [Fyne](https://fyne.io/) library. The search code is copied and adapted from [Andrew Healey's grup](https://healeycodes.com/beating-grep-with-go). I could, maybe should, be doing this in Dart and Flutter, but I happened to be using Go at the time. There's no indexing or anything fancy going on under the hood - what we have here is a text editor, grep and a simple user interface.

## Local File Storage

All the notes are stored as text files in a directory tree. The root is `.nincomp`.

The commonplace notes are stored in a subtree of that called `com`, which contains a number of directories, one for each book. The default book is called `Default`. Inside each book directory are text files, with the name of the file being a santitized version of the first line of each note. For example, if you had a note on cooking tips in the default book, it would be stored in a file called `.nincomp/com/Default/Cooking Tips.txt`.

The incremental notes are stored in a subtree called `inc`, which contains a number of directories, one for each book. The default book is called `Default`. Inside each book directory are directories for each year, and inside each of those, directories for each month. Each month directory contains text files for each day of the month. For example, if you made a note on January 5th 2023 in the default book, it would be stored in a file called `.nincomp/inc/Default/2023/01/05.txt`.

You can shadow the entire `.nincomp` directory tree in cloud storage, archive them in a [git](https://git-scm.com/) repository (which you can upload to a private github repository), or backup all the notes using, rsync or zip, for example, `zip -r <filename> .nincomp`. I use a little bash script to name the backup files after the date they were made, for example:

```bash
today=`date +%Y-%m-%d`
filename="nincomp$today.zip"
cd ~
zip -r $filename .nincomp
```

No tricksy or closed file formats here, no sir.

## TODO

- Better text editor (including spellchecking, found word highlighting, follow url, more visible caret, more keyboard shortcuts)
- The current search is very efficient, but case sensitive
- Support for moving text from `inc` to `com`, to facilitate short term to long term note workflow; maybe right-click popup 'copy selected text to commonplace book' ...
- Markdown support in `com` (fyne does have a markdown preview widget, but it's markdown support is, shall we say, basic). I *really* like WYSIWIG markdown editors like MarkText or Typora.
- More support for hashtags (eg tap on a hashtag to find notes containing it, insert hashtag from dict)
- Support for creating backups, or git, or cloud (Fyne has some cloud support)
- Many little quality-of-life tweaks like colors, keyboard shortcuts, and setting the font face and size.

## History

In the early 1990s, I wrote something called [Idealist](https://en.wikipedia.org/wiki/IdeaList). That grew out of the idea of merging a database manager and a text editor, and morphed eventually into a package of components that were used to build applications in the museum, archive, and library sectors. 250,000-odd lines of C and Tcl, but I, like many others, just used it to take notes.

Everytime I use a computer, I end up taking notes. Playing a game, developing a new app, doing finances, reading about the worldwide political horrorshows, building bicycles, planning a vegetable garden, following a tv series, reading a book, ... everything seems to generate notes.

So, I've spent decades trying different ways of doing that, trying different apps and methodologies, storing files all over the place and eventually losing them, or not being able to transfer them into the shiny new fashionable app. Idealist is too old, only runs on Windows, and I don't have the source, so I can't use that. After years of nagging at myself, I surrendered to the itch and am developing my own new app(s), to fit my needs, and trying really hard to do it in the simplest way possible. Because plain and simple are good.