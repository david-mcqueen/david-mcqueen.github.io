---
layout: post
title:  "Timelapse - 4 years of solumn faces"
date:   2019-03-28 21:13:00 +0000
categories: blog posts
---

## More vain than a Varicose vein

For someone who has a fairly-strong dislike for Social Media, I take a surprisingly vast amount of selfies. Every day I take a photo of myself. Every day it gets added to a photo album, claiming its role as the latest picture in a vast world of pictures - A new king ruling over those who've come before it, bringing with it another days worth of wisdom; Or an infant in a world of pictures who've been solidified in time for longer than they can remember? That is until the next day when I take another photo, unleashing another bout of confusion on the picture world. Oh I do enjoy it. Better than a mirror, as it doesn't argue back.

It all started back in November 2014 where for some long forgotten reason I decided to start a selfie-timelapse. And now 4 and a half years later I am still doing it - Each day can never be the day in which I stop doing it. It has turned into a project that with every passing day the project gets more and more complete. Although I will never see it complete.

I was originally using an app on my iPhone which worked great whilst I had an iPhone. However following my switch to Android, I struggled to find an equivilant app that I was happy with to take and stitch the photos together. They all felt just a little bit clunky

This left me with 2 problems, the first being requiring a program to stitch together over a thousand photos. A fairly trivial problem if we're going to be honest, those types of programs are fairly easy to come by. 

The second problem, though, exasperated the first in that the 2 phones took pictures in different dimensions and resolutions. The generic timelapse generators (at least the ones which I tried) would fail as they encountered the differing image sizes. I'm sure there are probably solutions out there which already handle this, but why not make my own!


## My own bespoke generic timelapse application
I'm currently learning GoLang, and conveniently there is a GoLang package of OpenCV, [GoCV](https://gocv.io/), allowing me to easily incorporate it into my application - I want an application which handles my specific use case, but is also able to handle other use cases as well.

The application itself is fairly trivial, it takes a couple of optional parameters;

```golang
var (
    dir = flag.String("dir", "./images", "Directory to find the images in")
    fps = flag.Uint("fps", 10, "Framerate")
    outputFile = flag.String("outputFile", "timelapse.avi", "Name of output file.")
    limit = flag.Uint("limit", 3000, "Maximum number of images to stitch")
)
```

Then does a couple of house keeping tasks; Checks the files exist; initialise the variables we need, such as image readers and video writers, and then most importantly gets ready to handle resizing of the images.

Because my pictures are all in different sizes, I need to have a constant image output size. As such, I store the dimensions of the first image in the provided directory. This will serve as an anchor, all images will be resized to be the same size as the first image.
```
newImage := gocv.IMRead(firstImage, gocv.IMReadAnyColor)

// Use the dimensions of the first image for the dimensions of all images
imageWidth, imageHeight := newImage.Cols(), newImage.Rows()
```

Then each of the images is read, resized, and written to the writer.

```golang
newImage := gocv.IMRead(dir + "/" + image.Name(), gocv.IMReadAnyColor)
defer newImage.Close()

resizedImage := gocv.NewMat()
defer resizedImage.Close()

// Specify the size that the image needs to be for it to be added to the output file
var pointy image2.Point
pointy.X = imageWidth
pointy.Y = imageHeight

gocv.Resize(newImage, &resizedImage, pointy, 0, 0, gocv.InterpolationLinear)

err := writer.Write(resizedImage)

```

The complete source code is on [GitHub](https://github.com/david-mcqueen/timelapse-stitcher).


### Collect yo' Garbage!
I initially had problems with my laptop seizing up when trying to run the entire program, with the resize functionality, over all of my photos. It turns out that I wasn't cleaning up my objects properly (Silly!). Each new image was loaded into memory and wouldn't be disposed of until the program itself had completly finished. Which it couldn't do because the earlier images were all keeping hold of the resources. The greedy buggers. 

But we can't blame them really can we.

I **thought** I had done all I needed to, however clearly I hadn't. Making one critical change, to move the logic for resizing and writing the image to the writer into into its own _function_, solved the problem. To understand why lets look at the [docs](https://golang.org/ref/spec#Defer_statements) where we have this statement;
> A "defer" statement invokes a function whose execution is deferred to the moment the surrounding function returns

Moving the resize and Write logic into its own function meant that each image would be processed, and then released as the surrounding function returned. The [fix](https://github.com/david-mcqueen/timelapse-stitcher/commit/f2666c4e9bf99281be535feebc6f92c077f0cb63) was incredibly simple, but made substantial performance improvements over my original code. Retrospectively, I imagine I could have stopped using _defer img.Close()_ and just called _img.Close()_ directly after the end of each loop (or whenever was appropriate for the object in question), but there is more than one way to skin a cat...


## Generating Gif

My bespoke generic timelapse application &copy; creates a video as its output, which isn't very web friendly. Using our favourite tool for converting video files, [FFmpeg](https://ffmpeg.org/), I simply converted it to a gif file.

```javascript
ffmpeg -i timelapse.avi -vf scale=128:-1 timelapse.gif
```
Stick a fork in ~~me~~ it...

## I try to smile!
Thats a lie. I don't. The scientist in me wants to keep my face as bland as possible so that the focus of the pictures is the ageing process, as opposed to my gawky _smile_. With that being said, a couple of weeks ago I was showing a couple of the images (pre-timelapse) to my dear mother and grandmother and they said that some of the images don't even look like me... I don't know if thats a good thing or not. They also said that I "_look very grumpy in these_". I guess I could at least try to smile. Maybe the next 4 years will be full of smiling photos...

Enough of me, here is what none of you have been waiting for; Over one thousand photos of me sporting a bland expression a gargoyle would be proud of...


![Timelapse](/assets/images/timelapse/timelapse.gif)

I had to reduce the gif down in both size and quality so that it was somewhat sensible - the original video was 109MB which I feel is slightly unreasonable to force visitors to be downloading. Heck I'd be happy with any visitors I can get, I hardly want to have a 100MB file creating a barrier of entry driving people **away** from these glourious ramblings.


## Enhancements to the program

This is only the beginning (hold the screams of excitement, please). I have a couple of ideas for things which I'd like to change or improve upon;
* Use OpenCV to detect the face and position it to be the middle of the image - hopefully reducing the jumpiness. Plus its more learning in OpenCV which is always fun
* Output to GIF directly, as opposed to outputting to a video and then needing a manual process to generate the gif
* Take additional parameters for filtering the source images based on date, so a timelapse over a specific period is created as opposed to the entire period
* Allow input of an image size, as opposed to just using the first image in the collection
* If I'm feeling extremely adventerous, then I might hook it up to something like AWS;
    * Store the pitcures in an S3 Bucket
    * Watch for when a certain number of new images have been added, automatically take them and create a new gif


The possibilities are endless...