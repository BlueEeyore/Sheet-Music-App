# Brief Overview

I want an app for android (and also apple if possible but not a priority) that allows you to upload pdfs of sheet music and then to view them, annotate them, and assort them.

# Key Features

- The app must let you upload pdfs and also have the option to upload an entire folder of pdfs.
- The app must let you link to a google drive folder or a local folder.
- The app must let you open the pdfs to view and annotate (details of this will be in the pages section)
- The app must let you create folders where you can move the pdfs that you have uploaded.
- The app must also let you create setlists where you can add pdfs from your library and change the order within the setlist.

# Pages in Detail

## Home/Library

Here you can see all of your pdfs. You can edit each of their names, delete them, add them to setlists, add labels, and move them into folders (there should also be an option to create said folders) by pressing a 3 dots symbol on their thumbnail. You can also long press to select one or more, and that gives you the option to move these to a folder, add them to a setlist, or delete them. Tapping on one of the pdfs in your library will take you to their viewing page. You can sort your scores by name (alphabetically, reverse alphabetically), last opened, label, last uploaded, and search.

You can add new pdfs to your library, and when pressing this button, you have the options of importing a pdf, or importing an entire folder (imports all of the pdfs in this folder at once and puts them in their own folder in your library). There is also the option to link a folder in your google drive or in your local storage.

Labels work so that you can create labels (with a name and colour), and add any score to a label that you have created. Then you have the option to sort by label, or remove particular labels from your sort.

## Viewing

When viewing each pdf, the pages themselves take up the entire screen by default. When tapping the left of the screen it takes you to the previous page and when tapping the right of the screen it takes you to the next page. This is done with a short page turning animation. Tapping the middle of the screen brings up options on the top of the page. These options include:
- Changing the display ratio of the pages (to fit the screen, to stay the same ratio as the original, custom, etc)
- Changing the view of the pages - options are one page at a time (default), 2 pages at a time (for this you either have both of them change to the next two pages each time you move forward eg pages 1&2 becomes 3&4, or you have them both move forward by 1 eg pages 1&2 becomes 2&3), or half page at a time (you view an entire page, but tapping to the next page only makes the first half of the page change, and tapping again makes the second half change. Mimick Lecturn for this feature).
- bookmarks (tapping on this opens up your bookmarks on the side of the screen and you can add one and name it, as well as jump to existing bookmarks)
- Metronome - tapping on this makes a metronome pop-up appear, where you can select the bpm (with option for tap tempo), select the beats per measure, have a few options for the sounds of beats, and have the options for audio, visual, or both (this is mimicking Lecturn's metronome). Visual means that the pulse will be around the border of the screen (just like in forescore).
- Tuner - opens up a tuner, with the option to play a tone.
- Links - selecting this opens your links on the side of the screen like for bookmarks. You can add links. When you add a link, it prompts you to tap where you want the source point to be, and then promts you to tap where you want the return point to be. Then, when viewing the score, the source point is visible as a large dot, and you can tap on it, and it will take you to the page that has the return point. The return point will then flash 3 times, but otherwise it is invisible (unless you have the links menu open). In other words, copy Lecturn for this.
- Crop - lets you crop the page, with options to undo, redo, reset, rotate, straighten, and apply changes to all.
- Notes - selecting this opens your notes for this pdf on the side of the screen (like with bookmarks). You can add notes here as well, and make folders for these notes. There should also be an option to make a global folder which makes the notes inside it visible in all pdfs.
- Arrange pages - takes you to an overview of all the pages in the pdf, and allows you to rearrange the order, remove pages, and add pages (via import from new pdf, another existing pdf in your library, or making a blank page)
- Eye comfort shield - tapping this makes the colour tone of the pages warmer (like the android eye comfort shield), and the exact tone and intensity can be edited in the settings (how much darker it makes the screen, and how much the colour tone changes)
- Export pdf - lets you download as pdf

Tapping on the page with your stylus opens up a "toolbar" of options for annotation at the bottom of the page:
- pen (default). Tappping on this lets you draw on the page with your stylus. Any annotations made are saved to this score on the app of course. Tapping on this while the pen is already selected lets you change colour, brush stroke size, and brush stroke type (variable width or strict width), like in samsung notes.
- favourite pens. These are customisable saved pen colours. The number of which can be changed in settings (options for 0-7 favourite pen colours).
- highlight. Exact same features as pen but lower opacity.
- favourite highlighters. Exact same features as favourite pens but with highlighter instead of pen
- eraser. Tapping this when already selected lets you choose between area eraser or stroke eraser, as well as the eraser size. Stroke eraser means that every individual stroke you touch will get erased, whereas area eraser will erase just the pixels that you touch. this should mimick the Samsung notes eraser.
- selector. Tapping this when already selected lets you choose between lasso and rectangle, and let you choose between stroke and area. Lasso will let you draw your own selection box, wherease rectangle creates a rectangle from where your pen went down to where it went up. Stroke means that all strokes that are even slightly in your selection box will be selected, and area means that only exactly what is in your selection box will be selected.
- Text box. This lets you create text boxes where you type words. Tapping this when already selected lets you change font size, style, and colour.
- Stamps - lets you place "stickers" of all the different existing sheet music notations (accidentals, dynamics, articulations, navigation, fingering, notes, rests, etc). These can obviously have their size altered after they are placed.
- undo/redo - this is self explanatory

The toolbar should be fully customisable in settings - you can add or remove tools from the toolbar.


### further features
- It should also be compatible with bluetooth pageturners
- The screen should not go dark (but make the time taken for the screen to go dark toggleable in the main settings)

## Recents

Exactly like the library page but just shows you your most recently accessed pdfs in order of how recent thy have been opened.

## Setlists

Has an option to create a new setlist which immediately takes you to the editing setlist page for an empty setlist.

You can also create folders for these setlists.

Tapping on a setlist takes you to its contents, which is an ordered set of scores in your library. Tapping on one of them will take you to the viewing page for that score, but when you reach the end of it, the next page is the next score in the setlist. There is also the option to edit this setlist both when viewing it and if you long press on it from the setlist index.

You can also merge setlists by long pressing and selecting multiple.

You can also duplicate setlists.

## Edit Setlist

Lets you move the scores in the setlist around, remove scores from the setlist, and add scores from your library. You can also add placeholders, which is just a name but not associated with a pdf. This will just show up as a blank page with the name that you assign it upon creation.

You can also duplicate the playlist.

## Settings

# Aesthetic

The aesthetic should be sleek, simple, and professional with colour palette of white and black, with this red: #E94A64, and navy.

# Final Note

In general, with regards to formatting and details for features, copy forescore, but consult me first.
