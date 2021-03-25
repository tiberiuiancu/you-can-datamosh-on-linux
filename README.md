Datamoshing made easy. 

## Requirements
`mosh.py` requires `ffmpeg` to be installed.

`vector_motion.py` and `style_transfer.py` depend on `ffedit` and `ffgac`, which can be downloaded from [ffglitch.org](https://ffglitch.org/)

# Effects

## i-frame removal
This type of glitch creates the transition effect. Example:

| Original | Moshed |
|:--------:|:------:|
| ![original_hand](https://user-images.githubusercontent.com/31802439/112060042-f3e42780-8b5c-11eb-8019-df4d06dd0d31.gif) | ![moshed_hand](https://user-images.githubusercontent.com/31802439/112060033-f181cd80-8b5c-11eb-9025-65064bbc6200.gif) |

    $ python mosh.py input.mp4 -s 40 -e 90 -o output.mp4
removes all the i-frames from the input video starting at frame 40 and ending at frame 90, and outputs the final result
to `output.mp4`

## p-frame duplication
Repeats a series of p-frames (aka delta frames), which can give a 'melting' effect. This type of glitch is triggered by the `-d` flag. Example:

| Original | Moshed |
|:--------:|:------:|
| ![original_dog](https://user-images.githubusercontent.com/31802439/112059335-0316a580-8b5c-11eb-98c8-3493969dd472.gif) | ![moshed_dog](https://user-images.githubusercontent.com/31802439/112060106-065e6100-8b5d-11eb-9670-4ad3bd9522cd.gif) |

    $ python mosh.py dog.mp4 -d 5 -s 165 -o moshed_dog.mp4

copies 5 frames starting at frame 165, then replaces all subsequent groups of 5 frames with the copied data (in this case until the video ends, as no `-e` flag was specified).

## Vector motion
While the previous effects copy and delete whole frames, this one changes the actual frame data. As explained in
[this article on ffglitch.org](https://ffglitch.org/2020/07/mv.html), you need to write a custom JavaScript file
that can change the frame data. `vector_motion.py` is just a wrapper for `ffedit` and `ffgac` and makes moshing
possible through only one command.
Example:

    $ python vector_motion.py input.mp4 -s your_script.js -o output.mp4


### Now also with Python!

If you prefer to use python to glitch the frames instead, you can also specify a python script for the `-s` option.
The script must contain a function called `mosh_frames` that takes as argument an array of frames (warning: some of the frames
might be empty), where each non-empty frame represents a 3D array of shape (height, width, 2). The function should
return an array containing the modified vectors. 

`horizontal_motion_example.py` contains the equivalent python script of the js script presented in the
[ffglitch tutorial](https://ffglitch.org/2020/07/mv.html).

`average_motion_example.py` is the equivalent of this [ffglitch average motion tutorial](https://ffglitch.org/2020/07/mv_avg.html)
using numpy. Neat!


## Style transfer
I call style transfer combining the motion vectors of two videos (by simply adding them together). For example,
applying the vector motion data of a person talking to a video of clouds can make it look as though the clouds
are talking. 

This script can also extract motion vector data from a video and write it to a file, or read motion data from file and
apply it to a video.

Examples:

    $ python style_transfer.py -e person_talking.mp4 -t clouds.mp4 output.mp4

extracts vector data from `person_talking.mp4`, transfers it to `clouds.mp4` and outputs the video to `output.mp4`.

| Extract style from | Transfer style to | Result |
|:------------------:|:-----------------:|:------:|
| ![clouds](https://user-images.githubusercontent.com/31802439/112489124-70a21c00-8d7e-11eb-8640-6817a46602ca.gif) | ![trees](https://user-images.githubusercontent.com/31802439/112489146-74ce3980-8d7e-11eb-9091-999fbb98552c.gif) | ![ct](https://user-images.githubusercontent.com/31802439/112489221-86afdc80-8d7e-11eb-9a51-14d91ec7cdfa.gif) |


    $ python style_transfer.py -e person_talking.mp4 vectors.json

extracts the vector data from `person_talking.mp4` and outputs it to `vectors.json`.


    $ python style_transfer.py -v vectors.json -t clouds.mp4 output.mp4

loads vector data from `vectors.json`, transfers it to `clouds.mp4` and outputs the video to `output.mp4`.
