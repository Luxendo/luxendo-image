# Luxendo Image

In the following, the "Luxendo Image" (`.lux.h5`) format is defined and explained.
Its purpose is to provide an open, transparent image format for 3d SPIM images acquired by Luxendo microscopes, as well as for output images from further processing / analysis.
We designed this format to contain all the relevant acquisition, viewing, and processing metadata in a clear and flexible form.
Luxendo Image is based on the widely used, open HDF5 (`.h5`) format, to achieve broad compatibility.

## Goals

The main goals for the Luxendo Image format are:

* Stability and long-term compatibility
* Store image data corresponding to different time points and channels in *separate* `.lux.h5` files, to keep file sizes as small as possible and facilitate the handling of the files.
* Easy linking to the *same* `.lux.h5` files from *different* "master" files (e.g. Fiji BigDataViewer) to provide compatibility with different tools / viewers while avoiding duplication of image data.
* Enable everyone to load / output Luxendo Image files into / from their own software and custom processing / analysis pipelines.
* Metadata:
  - Clearly distinguish between "acquisition" and "derived" metadata to transparently keep track of metadata, e.g. when multiple images are fused.
  - Use the same form of metadata for both newly acquired images as well as processed images so that already processed images can be fed back into pipelines for further processing.
  
## Structure

A Luxendo Image (`.lux.h5`) file is a regular `.h5` (HDF5) file that has a defined internal structure and metadata.

### Image Data

The image data is stored in a h5 dataset called `"Data"`, as 3d array with 16-bit unsigned-integer (`uint16`) elements.

If there are multiple datasets in one `.lux.h5` file, then they correspond to different resolution layers of a resolution pyramid:
* `"Data"`: dataset containing the highest-resolution image data.
* `"Data_2_2_2"`: dataset containing image data that is downsampled in each dimension by a factor 2.
* `"Data_3_3_3"`: image data downsampled by factor 3 in each dimension.

and so on.

The order of the downsampling factors is: `Data_<width>_<height>_<depth>`, referring to the width, height, and depth of the image.
And they are always *integer* factors.

The highest-resolution image data is typically chunked with chunk size `64*64*64`, whereas lower-resolution data may be chunked with chunk size `32*32*32`.

### Metadata

A `.lux.h5` file has a h5 dataset called `metadata` that contains JSON data `{"processingInformation": ...}`.
The entry `"processingInformation"` has all the relevant metadata (in JSON form).
This metadata is divided into "acquisition" metadata and "derived" metadata.

Its high-level structure is:

```
"processingInformation": {

  < ... all "derived" metadata (e.g. transforms) ... >

  "acquisition": [
  
     < ... all "acquisition" metadata (i.e. microscope settings, geometry) ... >
  
  ]

}
```
In this structure:
* All the acquisition settings are inside the `"acquisition"` field.
* Metadata outside `"acquisition"` is derived / calculated from the `"acquisition"` metadata or stems from processing (e.g. transforms from image registration).
* The `"acquisition"` field (list) can have *multiple* items corresponding to different acquired images, e.g. when the given Luxendo Image contains image data that results from a fusion of multiple images.

Note that the same form of `"processingInformation"` is used for both newly acquired images as well as processed / fused images. Thus, already processed images can be fed back into a processing pipeline for further processing.

The concrete form of the `"processingInformation"` metadata is the following (values are just examples):

```
"processingInformation": {
    "version": "1.0.0",
    "sources": ["Luxendo MuVi-SPIM"],
    "contains_beads": false,
    "time_point": "00000",
    "channel": "1",
    "stack": "0",
    "objective": "left",
    "camera": "left",
    "time_stamps": ["2020-06-19_144019"],
    "voxel_size_um": {
        "width": 0.40625,
        "height": 0.40625,
        "depth": 1
    },
    "image_size_vx": {
        "width": 2048,
        "height": 2048,
        "depth": 441
    },
    "affine_to_sample": [
        {
            "matrix": [
                [1, 0, 0],
                [0, 1, 0],
                [0, 0, 1]
            ],
            "translation": [-1023.5, -1023.5, -220],
            "type": "same-stack:center"
        },
        {
            "matrix": [
                [-0.40625, 0, 0],
                [0, 0.40625, 0],
                [0, 0, 1]
            ],
            "translation": [0, 0, 0],
            "type": "same-stack:geometry"
        },
        {
            "matrix": [
                [1, 0, 0],
                [0, 1, 0],
                [0, 0, 1]
            ],
            "translation": [150, 3200, 380],
            "type": "inter-stack:translation"
        },
        {
            "matrix": [
                [1, 0, 0],
                [0, 1, 0],
                [0, 0, 1]
            ],
            "translation": [-36, 0, -297],
            "type": "inter-stack:rotation-offset"
        },
        {
            "matrix": [
                [-0.86602540378443871,-0, -0.49999999999999994],
                [0, 1, 0],
                [0.49999999999999994, 0, -0.86602540378443871]
            ],
            "translation": [246, 0, 488],
            "type": "inter-stack:rotation"
        }
    ],
    "detection_directions": [
        [0, 0, 1]
    ],
    "acquisition": [
        {
            "microscope_type": "MuVi-SPIM",
            "serial_number": "10047",
            "embedded_version": "3.0.0",
            "edits": [],
            "contains_beads": false,
            "time_point": "00000",
            "channel": "1",
            "stack": "0",
            "objective": "left",
            "camera": "left",
            "stage_positions": [
                {
                    "name": "x",
                    "start_um": 3200,
                    "end_um": 3200,
                    "movement": {
                        "direction": [0, -1, 0],
                        "offset_um": [0, 0, 0]
                    },
                    "type": "linear"
                },
                {
                    "name": "y",
                    "start_um": -150,
                    "end_um": -150,
                    "movement": {
                        "direction": [1, 0, 0],
                        "offset_um": [0, 0, 0]
                    },
                    "type": "linear"
                },
                {
                    "name": "z",
                    "start_um": 160,
                    "end_um": 600,
                    "movement": {
                        "direction": [0, 0, -1],
                        "offset_um": [0, 0, 0]
                    },
                    "type": "linear"
                },
                {
                    "name": "r",
                    "start_deg": 150,
                    "end_deg": 150,
                    "movement": {
                        "direction": [0, 1, 0],
                        "offset_um": [-246, 0, -488]
                    },
                    "type": "rotation"
                }
            ],
            "image_plane_vectors": {
                "cam_left_to_right": [-1, 0, 0],
                "cam_top_to_bottom": [0, 1, 0]
            },
            "refractive_index": 0,
            "detection": {
                "wavelength_nm": 0,
                "numerical_aperture": 0.80000000000000004,
                "magnification": 16,
                "cam_pixel_size_um": {
                    "width": 6.5,
                    "height": 6.5
                },
                "sensor_size_px": {
                    "width": 2048,
                    "height": 2048
                },
                "crop_offset_px": {
                    "left": 0,
                    "top": 0
                }
            },
            "illuminations": [],
            "time_stamps": ["2020-06-19_144019"]
        }
    ]
}
```

In the following, the different fields are explained:

#### "Derived" metadata

This metadata is derived (calculated) from the `acquisition` metadata.

* `version`: Version number of the Luxendo Image format.
* `sources`: List of sources (software) that generated / contributed to the given Luxendo Image file.
* `contains_beads`: Whether the image contains beads (relevant e.g. for image registration).
* `time_point`: Name of the time point corresponding to the image.
* `channel`: Name of the "channel". A channel usually refers to a combination of different settings such as illumination and detection wavelengths and other settings that do *not* decide the positioning of the sample.
* `stack`: Name of the "stack". A stack usually refers to a specific positioning of the sample during acquisition.
* `objective`: Name of the detection objective through which the given image was acquired.
* `camera`: Name of the camera that acquired the image data.
* `time_stamps`: Date and time of the acquisition (corresponding to the *first* frame of the 3d image). There can be multiple time stamps, e.g. in case of a fused image that was created from multiple images, with the time stamps corresponding to the contributing original images.
* `voxel_size_um`: Physical size of the voxels in sample space, in micrometers. `"depth"` is the shortest spacing between the image planes (perpendicular to planes). Important things to note:
  - This voxel size is *also* contained in the (first applied) scaling part of the transform given by `affine_to_sample` (see below). This is because, when opening the image with a viewer/software that is unable to apply the `affine_to_sample` transform, it should still be able to show the data with the correct voxel size, read from `voxel_size_um`. 
  - On the other hand, any viewer or processing software that *can* apply the `affine_to_sample` transform should *ignore* `voxel_size_um` and use `(1,1,1)` as voxel size instead, and then automatically get the correct voxel size from applying `affine_to_sample`, which contains the scaling.
  - Consequently, software that outputs Luxendo Image files should not only write the voxel size to "voxel_size_um", but *also* include the corresponding scaling transform as first transform in `affine_to_sample`: a diagonal matrix with the voxel sizes in `voxel_size_um` on the diagonal.
* `image_size_vx`: The image size in each dimension, in number of voxels.
* `affine_to_sample`: The affine (scaling, rotation, shear, translation) transform that brings the image data to the "sample space", the space permanently connected with the sample stage. Note that for a rotation stage, the sample space may be rotated with respect to the "microscope space" (reference frame of the microscope, which can be thought of as axes along the edges of the microscope "box"), e.g. for the Luxendo MuVi SPIM system. For zero sample-stage rotation and zero sample-stage translation, the "sample space" and the "microscope space" are aligned. `affine_to_sample` can be a list of multiple affine transforms that are to be combined, where the ordering is such that the first transform in the list is the first one applied, and the other transforms follow in the given order. Each affine transform has a `matrix` field containing the *rows* of the affine-transform matrix, and a `translation` field with the 3d translation vector. A field `type` is used to label the transform -- e.g. `"rotation"` if the transform corresponds to a rotation. The `translation` vectors are also used to position the image stack at the correct physical location in sample space.
* `detection_directions`: The direction of detection (axis of detection objective) as vector in 3d in the current space of the image data. Initially, after acquisition, this space is the "pixel space" (space in which the image data from the camera is originally stored), and so the detection direction is parallel to the vector `(0, 0, 1)`. There can be multiple detection directions, e.g. in case of a fused image consisting of multiple original images with given original detection directions.

#### Acquisition metadata

Inside the `acquisition` field, we have the following metadata fields:

* `microscope_type`: The type of microscope from which the acquired image data stems (e.g. "Luxendo MuVi-SPIM").
* `serial_number`: The serial number of the microscope.
* `embedded_version`: Version number of the embedded system running on the microscope.
* `edits`: In case there were edits to the `acquisition` metadata, this field holds a list of origins and times of the edits.
* `contains_beads`, `time_point`, `channel`, `stack`, `objective`, `camera`: These have the same meaning as in the "derived" metadata. They appear here again since the image could be a fused image consisting of multiple original images, in which case in the "derived" metadata section we would e.g. have `"camera": "Fused-Left-Right"`, while in the `acquisition` section we would have one set of metadata containing `"camera": "Left"` and another set containing `"camera": "Right"`.
* `stage_positions`: Start and end positions (in micrometers, or degrees in case of rotation) for each stage-movement direction, together with the name of the respective direction. Each movement direction and offset are specified as vectors in 3d "microscope space" (reference frame of the microscope).
* `image_plane_vectors`: The two vectors in 3d microscope space that "span" the imaging plane (plane of the light sheet) along the image "edges" of the camera. These two vectors describe the orientation of the imaging plane with respect to the microscope reference frame, and thus with respect to the stage-movement directions. This flexibly defines the geometrical arrangement of the microscope.
  - `cam_left_to_right`: Vector along the upper edge of the camera image, from left to right.
  - `cam_top_to_bottom`: Vector along the left edge of the camera image, from top to bottom.
* `refractive_index`: The refractive index of the medium that encloses the sample.
* `detection`: Detection-related parameters, such as:
    - `wavelength_nm`: Detection wavelength (in nanometers).
    - `numerical_aperture`: Numerical aperture.
    - `magnification`: Effective magnification.
    - `cam_pixel_size_um`: Size of a pixel on the camera sensor (in micrometers).
    - `sensor_size_px`: Size of the camera sensor in pixels.
    - `crop_offset_px`: Offset in case the image is cropped (in pixels).
* `illuminations`: Illumination-related parameters.
* `time_stamps`: Acquisition date and time.
  

### The "main" file

A "main" file `main.lux.h5` links to all the datasets in the Luxendo Image files (other `.lux.h5` files), including the `metadata` dataset (containing `processingInformation`). It contains separate links to all the different resolution levels of the image data. It can link to multiple channels, time points, cameras, stacks, etc.

A `main.lux.h5` file has a nested structure (example):

```
timepoint_First
  channel_First
    view_First
      res_0
        data
        metadata
      res_1
        data
      res_2
        data
                
timepoint_Second
  channel_First
    view_First
      res_0
        data
        metadata
      res_1
        data
      res_2
        data
```

Within a time point `timepoint_<name>`, there can be multiple channels `channel_<name>`, which in turn can contain multiple views `view_<name>`.
At the deepest nesting level there are the different resolutions, going from highest (`res_0`) to lowest (`res_N`) resolution.
In each resolution, there is a link `data` to the corresponding image data in a h5 dataset in a *different* `.lux.h5` file.
The *highest* resolution `res_0` also has a link `metadata` to the corresponding `metadata` h5 dataset in the other `.lux.h5` file.

The `main.lux.h5` file resides in a folder (usually named with a date and time stamp) corresponding to the experiment in which the images were acquired.
It links to the raw (and processed) images (`.lux.h5`), which are also contained in this folder.
To move or copy the images to a different location, this folder should be moved/copied as a whole, to not break the links in the `main.lux.h5` file pointing to the image data.

The individual image files (`.lux.h5`) should also not be renamed manually or moved to different subfolders inside the experiment folder, since this would break the links as well.
    

