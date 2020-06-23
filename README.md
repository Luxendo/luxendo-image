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
* `"Data222"`: dataset containing image data that is downsampled in each dimension by a factor 2.
* `"Data333"`: image data downsampled by factor 3 in each dimension.

and so on.

The highest-resolution image data is typically chunked with chunk size `64*64*64`, whereas lower-resolution data may be chunked with chunk size `32*32*32`.

### Metadata

A `.lux.h5` file has a field called `"processingInformation"` that contains all the relevant metadata in `JSON` form.
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
            "translation": [246, 0 488],
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
            "sensor_size_px": {
                "width": 2048,
                "height": 2048
            },
            "crop_offset_px": {
                "left": 0,
                "top": 0
            },
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
                }
            },
            "illuminations": [],
            "time_stamps": ["2020-06-19_144019"]
        }
    ]
}
```





