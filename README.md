rode-rti-builder-python
=======================

This is a proof-of-concept Python utility to process RTI input.

RTI: http://culturalheritageimaging.org/Technologies/RTI/

# Notes after hackathon:

This project was thrown together over two days by Karl and John. We were
unable to completely join the image processing code to the GUI in the end 
because of linking problems, but the separate parts do appear to function.

We timed the image processing side at about 40s total real time for all
work, so about 5x to 10x faster than RTIBuilder, the program this thing
is supposed to improve upon.

Thoughts:

- We used pyqt5 for the GUI, which may not be the right choice. The Debian
  pyqt5 package seems to be python3 only, but sadly libvips still just makes a
  python2 module. 

  I looked into moving the vips python module to python3, but it seems harder
  than you'd think. The buffer system has been redone for python3, so that
  would need to be reimplemented, all (?) strings are now `wchar_t`, and
  libvips `configure` does not detect python3 correctly (you need to add a few
  '()' to the print statements in the path detector program that configure
  writes).

  Additionally, Karl had spent only a relatively small amount of time with
  pyqt5 before the hackathon and had no relevant pre-existing code to reuse. 

  I have rtiacquire, a pygtk2 program, which already has quite a bit of the
  functionality we need. This might have made a better basis for work, even
  though pygtk2 is rather poor compared to pyqt5.

  pyqt4 would be another option. 

- The lp file generation seems not to match the numbers produced by
  RTIBuilder. I only guessed the meaning of the columns, this needs to be
  looked into.

- We generally load images to memory, even when this is not necessary. For
  example, `make_average.py` could stream images, but does not. 

  Fixing this is not easy unfortunately and needs some changes to libvips. For
  compatibility reasons, the vips7 jpeg loader interface can't load 
  sequentially, even if you specify "rs" as the open mode. See comments in
  `im_jpegs2vips.c`.

- I wanted to keep libvips dependencies out of Karl's GUI, so the three
  image-processing functions work in terms of filenames rather than VImage.
  This results in the same image being loaded several times. 

  We could fix this by adding libvips as an import to the GUI and save
  (probably) several seconds.

  If we moved ptmfit in process we could save some more time. 

- The GUI needs some obvious things:

    * A circle drawn on the average image showing the position of the detected
      ball and letting you adjust it.

    * The positions of the detected highlights need to be drawn too.

    * The highlight detector can fail to find a highlight in an image. This
      message is currently just printed, it should cause the GUI to display a
      warning.

    * Automatically sanity-checking the images for obvious errors would be very 
      useful. 

    * The GUI needs to be able to pan and zoom on the average image. Making
      our own high-quality image display widget would be a lot of work. 

    * Settings needs to be automatically saved and restored between sessions. 



