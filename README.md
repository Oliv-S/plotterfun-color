# Plotterfun-color

A collection of algorithms for turning images into color vector art.

![linedraw.png](/examples/linedraw.png)

Based on [mitxela/plotterfun](https://github.com/mitxela/plotterfun).

## Usage

When you load a picture, you can drag the preview image around and zoom with the scroll wheel.

The size is limited to 800px when you first load the image, but you're welcome to make it as big as you like. This is mostly because some algorithms have hard-coded defaults (like blur radius or border size) which work best with images of about 800px wide. However, if you're trying to produce a Stipple or Delaunay image with a huge number of stipples, the integration will not work very well if the polygons are too small, so making the image bigger can improve the results.

Each slider has a text box next to it, where you can type in an arbitrary number. Absurd numbers may crash the algorithm, but in the worst case scenario refreshing the page will fix everything. You're welcome to attempt a million stipples if it takes your fancy. 

## Plotting

When you're happy with the vector image, download the SVG and open it in Inkscape. Resize it (change units to mm or inches) then head to Extensions > Export > Plot. 

## Developing new algorithms

Each algorithm is in its own .js file. The `helpers.js` file can be imported to simplify things. The algo first sends a message describing the controls it needs, which usually starts with `defaultControls` (brightness/contrast/inverted etc). In addition to sliders, checkboxes and combo boxes are possible.

The worker is then sent a message with configuration (describing the state of the controls) and the imagedata. If using the default controls, `pixelProcessor` from `helpers.js` will return a function that lets you get the darkness of a pixel after the brightness/contrast has been applied.

Each time a control is changed, the algorithm is killed and restarted. If the algorithm is complex, you can post messages to the main thread with status information. To send vector data, use `postCircles` or `postLines` to send paths, or lists of paths.

For complex algorithms, specifying `noRestart: true` on a control will stop it from killing and restarting the worker. However, since workers won't receive another message until they've finished working, you will need to periodically stop and check for messages to make use of this. The approach I took in stipple.js was to wrap chunks of work in `setTimeout()` calls and make the main rendering an `async function` that can `await` them. You might describe this as a method-of-least-ugliness.

I strongly recommend reading the code for the existing algorithms, I've tried to make things as clear as possible.

### About webworkers

For "security" reasons, browsers refuse to run webworkers from files loaded straight from your disk. As a hacky fallback, plotterfun will detect this happening and append the script to the document, so that simple algorithms can still be developed this way. Complex algorithms will either freeze the interface while they run, or if they're asynchronous, risk running repeatedly since they cannot be killed properly.

It's much better to host the files locally for development, using either node's `http-server` or `python -m http.server`. Be aware that browsers aggressively cache webworkers, it's **essential** to have the developer tools open and "Disable Cache" ticked, or you will not have any fun.

# Examples
![squiggle.png](/examples/squiggle.png)
![squiggle_details.png](/examples/squiggle_details.png)
![linedraw.png](/examples/linedraw.png)
![linedraw_details.png](/examples/linedraw_details.png)
![spiral.png](/examples/spiral.png)
![springs.png](/examples/springs.png)
![subline.png](/examples/subline.png)
![subline_details.png](/examples/subline_details.png)
