# Rt 106 Programming Reference

This document accompanies the higher-level descriptions found in the [Algorithm SDK](ALGORITHM_SDK.md) and the [Application SDK](CUSTOM_APPLICATION_SDK.md), and provides more detailed programming information.

This document describes the following:
* JSON Metadata for Integrating Algorithms
* Datastore Python API
* REST API Provided by rt106-server
* AngularJS Services Provided by rt106-app
* AngularJS Factories Provided by rt106-app


## JSON Metadata for Integrating Algorithms


The metadata for an integrated algorithm is documented using a JSON format in _rt106SpecificAdaptorDefinisions.json_.  Several examples are provided in _rt106-algorithm-template_, _simple-region-growing_, and _wavelet-nuclear-segmentation_.  Please refer to these as a starting point for your own algorithm's metadata.

Here is a description of the individual algorithm metadata fields:

* __name__ - should be lower case, connected by dashes.
* __version__ - can be according to your own versioning system, possibly using major_minor_patch versions e.g. v1_0_0.  Underscores should be used.
* __queue__ - concatenate name and version, separated by two underscores, e.g. "__"
* __parameters__ - this section includes both input data and input parameters, represented in a JSON object, each tagged with the name of a parameter (or input).  The parameters section also includes a _required_ field that lists those parameters that are required by the algorithm.  Those that are not required may be displayed in the user interface along with a control for the user to specify whether or not the parameter should included.  Sub-fields of each parameter include:
  * _label_ -- a name that will be displayed in the user interface.
  * _description_ -- brief documentation on the parameter.
  * _type_ -- controls what type of input control is displayed, supported types include:  _number_, _boolean_, _text_, _series_, (currently-displayed DICOM image series), _voxelIndex_ (coordinates selected from current image).
  * _default_ -- starting value of the parameter if it is not edited by the user.
* __results__ -- describes each output of the algorithm.  Subfields are the same as for parameters, except there is no _default_ field.
* __result_display__ -- describes how output images should be displayed.  _NOTE:  This feature is limited and experimental.  It may be extended and improved based on user priorities._  This is a JSON structure having the following fields:
  * _grid_ -- the shape of an image grid represented as [rows,columns].
    * _shape_ -- number of rows and columns, represented as a list of two numbers, e.g. [2,1].  _NOTE:  Only the values 1,2 are currently implemented._
    * _columns_ -- relative sizes of each column.  _Not yet implemented._
    * _rows_ -- relateive sizes of each row.  _Not yet implemented._
  * cells -- list of descriptions of the "cells" that contain image viewers.
    * _row_ -- which row, with first one numbered zero.
    * _column_ -- which column, with first one numbered zero.
    * _cellType_ -- "image" (DICOM) or "pathologyImage"
    * _source_ -- either "context" (input) or "result" (output)
    * _parameter_ -- name of input parameter or result, defined above
    * _cellDisplayMode_ -- "background" or "overlay"
    * _imageProcessing_ optional parameter to process the image before displaying it.  The only currently supported option is "toBinaryImage".
    * _properties_ -- optional parameter to set the color of a monochrome image, e.g. "properties" : { "color" : "rgb(255,51,51)" }
    * _controls_ -- optional parameter to assocate a color picker and opacity slidebar with an image, e.g.: "controls": { "opacity": "Mask Opacity: ", "color": "Mask Color: " }.  The value of each element is the label to be displayed in the user interface.
* __api__ -- not yet used
* __doc__ -- not yet used
* __classification__  -- hierarchical classification of the algorithm e.g. "segmentation/cellular/nuclei"  Can be used in the app for filtering the algorithm list to certain types.

## Datastore Python data API
These functions are available for the short Python script that you need to write that marshalls data from the Rt 106 datastore to your algorithm and stores generated results back to the datastore.

The Rt 106 data API has been written with an initial focus on medical imaging / radiology data and pathology / microscopy data, but has been designed so that additional data types can be easily added later.

Rt 106 supports a common data API that is independent of storage mechanism.  The initial release of Rt 106 includes a "Datastore-Local" implementation which stores data files in a local directory structure.  In the future, alternate data storage mechanisms can be built, including direct storage in Amazon S3 or a PACS system.

An important concept for Rt 106 datastores is that a __path__ is a storage location within a datastore, but the format of the path may be specific for the type of datatore.  Therefore, code that calls the data API should never presume to know the structure of paths; the path should always be obtained from an API call to the datastore that returns a path.

### Medical Imaging / Radiology Data API Calls
* __datastore.retrieve_series( series_path, output_dir )__
  * description:  Retrieve the DICOM series represented by the provided path and store it (within the algorithm container) in the specified output directory.
  * parameters:
    * series_path:  Path name where the series is stored in the datastore.  (This should have been received as part of the incoming algorithm request message.)
    * output_dir:  Local directory within the algorithm's container where the series should be stored as a tar file.
  * return value:  HTTP-standard status code (i.e. 200 means OK.)
* __datastore.get_upload_series_path( input_path )__
  * description:  Given the path to a DICOM series that will be input to the algorithm, return a path where the algorithm result (another DICOM series) should be loaded.
  * parameter:
    * input_path:  Path name where the input series is stored in the datastore.
  * return value:  Path name where an output DICOM series should be stored, or an error code.
* __datastore.upload_series( series_path, input_dir, force=False )__
  * description:  Write a DICOM series stored in the local container to the specified path in the datastore.
  * parameters:
    * series_path:  Path where the series is to be uploaded to the datastore.
    * input_dir:  Directory in the local Docker container where the series is already located.
    * force:  Whether to overwrite a previously uploaded series at the same path.
  * return value:  A JSON structure containing a response code and other information.  If there is already a series in series_path, the upload will not be performed and an error is returned.
* __datastore.get_radiology_result_path( input_path, execid, step, tag )__
  * description: Return the storage path for the given result image (i.e. generated by an algorithm or manual annotation, rather than by acquisition equipment), where the image is specified by execid, pipeline step, and output tag.
  * parameters:
    * input_path: path to the input image
    * execid: name of pipeline execution.  A pipeline may be run multiple times with different parameter settings, so that results from multiple runs can be managed and compared.
    * step: name pipeline step or algorithm.
    * tag: name for one of the outputs from the algorithm.
   * return value: Path where the corresponding image can be stored or retrieved, or an error code.
* __datastore.radiology_result_execution_exists( input_path, execid )__
  * description: Return a boolean to indicate whether the input path is available rather than already occupied by another series.
  * parameters:
    * input_path: path where we would like to upload an image
    * execid: name of the pipeline execution.
  * return value: True if the input_path is available, otherwise False.
* __datastore.get_primary_data_formats( patient, exam, element )__
  * description:  Return the list of data formats available for a patient / exam / data element.
  * parameters:
    * patient: patient ID
    * exam: exam ID
    * element: element ID (e.g. "Imaging," etc.)
  * return value: JSON structure containing a 'formats' element having a list of available formats.
* __datastore.get_primary_data_path_single_file( patient, exam, element, format )__
  * description:  Return the path and filename for a single-file patient / exam data element.
  * parameters:
    * patient: patient ID
    * exam: exam ID
    * element: element ID (e.g. "Imaging," etc.)
    * format: format ID
  * return value: JSON structure containing a 'path' element and a 'filename' element.
* __datastore.get_derived_data_root_path( patient, execid, analytic )__
  * description:  Return the root path for a patient, pipeline execution, and specific analytic.  This may be where some metadata may be stored.
  * parameters:
    * patient: patient ID
    * execid: pipeline execution ID
    * analytic: analytic that was run
  * return value: JSON structure containing a 'path' element.
* __datastore.get_derived_data_formats( patient, execid, analytic, result )__
  * description:  Return the list of data formats available for a patient / pipeline execution / analytic / result data element.
  * parameters:
    * patient: patient ID
    * execid: pipeline execution ID
    * analytic: analytic that was run
    * result: the result for the execution of the analytic
  * return value: JSON structure containing a 'formats' element having a list of available formats.
* __datastore.get_derived_data_path_single_file( patient, execid, analytic, result, format )__
  * description:  Return the path and filename for a single-file result data element.
  * parameters:
    * patient: patient ID
    * execid: pipeline execution ID
    * analytic: analytic that was run
    * result: the result for the execution of the analytic
    * format: format ID
  * return value: JSON structure containing a 'path' element and a 'filename' element.

### Pathology / Microscopy Data API Calls
* __datastore.get_pathology_primary_path( slide, region, channel )__
  * description: Return the storage path for the given "primary" image (i.e. acquired in lab rather than algorithm-generated), where the image is specified by slide, region, and channel.
  * parameters:
    * slide: name of the slide
    * region: name of the region on the slide
    * channel: acquisition channel, which may be a wavelength of light or a logical name, such as biomarker.
  * return value: Path where the corresponding image can be stored or retrieved, or an error code.
* __datastore.get_pathology_result_path( slide, region, pipelineid, execid)__
  * description: Return the storage path for the given result image (i.e. generated by an algorithm or manual annotation, rather than by acquisition equipment), where the image is specified by slide, region, pipelineid, and execid.
  * parameters:
    * slide: name of the slide
    * region: name of the region on the slide
    * pipelineid: name of a "pipeline" in the sense of a run of the analysis pipeline -- A set of algorithms may be run multiple times with different parameter settings.  The pipeline is a name associated with a run so that results from multiple runs can be managed and compared.
    * execid: name or ID of a run of one of the steps within a set of algorithms.
   * return value: Path where the corresponding image can be stored or retrieved, or an error code.
 * __datastore.retrieve_multi_channel_pathology_image( input_path, output_dir )__
   * description: Retrieve all the images (multiple channels) for a given slide region.
   * parameters:
     * input_path: path where the region data is stored
     * output_dir:  local directory within the container where the downloaded images should be stashed
   * return value: status code

### Data API Calls that Pertain to Any Type of Data
* __datastore.get_instance( path, filedir, filename, format )__
  * description: Retrieve a data instance from the datastore and save it in a file in the local container.
  * parameters:
    * path: location of the data in the Rt 106 datastore
    * filedir: local directory into which to store the data file
    * filename: local file into which to store the data
    * format: the format of the data (e.g. 'dicom', 'tiff16', 'csv')
  * return value: status code
* __datastore.post_instance( path, filedir, filename, format, force )__
  * description: Store a file from the local container to the datastore.
  * parameters:
    * path: location of the data in the Rt 106 datastore to store the data
    * filedir: local directory containing the data file
    * filename: local file containing the data
    * format: the format of the data (e.g. 'dicom', 'tiff16', 'csv')
    * force: If false, trying to overwrite data that is already in the datastore returns an error.  If true, overwrite is allowed without an error.
  * return value: JSON structure with details, or an error code

## REST API Provided by rt106-server
Efforts have been made to define a stable API even for the initial release of Rt 106.  Over time, we expect new API calls to be added, especially to support additional data types.  Our goal is minimize changes to APIs after they have been released.  If there is ever a need for drastic changes to the API, we would advance the API version to /v2, and /v1 would still be provided to support existing applications.

In the descriptions below, a term in the REST API preceded by a colon indicates that a specific name should be provided in that position.  (This follows the conventions used by Node.js Express for REST calls.)

### REST API calls related to algorithm execution

* __/v1/execution (POST)__
  * Description:  Request an algorithm execution.
  * Returns:  Status of successfully receiving and queuing the request.
  * Payload: JSON structure specifying the algorithm to run and its context, such as this example:
    ```json
    {
      "analyticId": {
        "name":"algorithm-template--v1_0_0",
        "version":"v1.0.0"
      },
      "context":{
        "inputSeries":"/Patients/pat004/Primary/Imaging/studyD/spine_Anonymous360_PURE",
        "seedPoint":[0,0,0],
        "threshold":0
      }
    }
    ```

* __/v1/executions (GET)__
  * Description:  Query the rt106-server's in-memory list of algorithm executions.
  * Returns:  Array of JSON objects describing the history of algorithm executions.  Each record includes the analytic name, an execution ID, time stamps for the request and completion, all the input parameters and result data.  For data that is stored in the datastore (i.e. image files) the URI is listed rather than the actual file contents.

* __/v1/queryExecutionList (GET)__
  * Description:  Tells rt106-server to restore its in-memory algorithm execution list from the database.  This is typically called on system startup.
  * Returns:  A status string.

* __/v1/analytics/evaluation (POST)__
  * Description:  Write the user's evaluation of an algorithm to the database.
  * Payload: A JSON structure containing user feedback about an algorithm.  Fields include execution ID, an evaluation value (typically a number on a scale provided to the user) and a comment string.

### REST API calls related metadata on algorithms

* __/v1/analytics (GET)__
  * Description: Query the list of running analytics.
  * Returns: JSON dictionary where each key is an analytic name.

* __/v1/analytics/:analytic (GET)__
  * Description:  Query for a specific algorithm by name.
  * Returns: A JSON structure having the name and version of the algorithm if it exists, or an error code if it does not exist.

* __/v1/analytics/:analytic/api (GET)__
  * Description: Query the API for the given algorithm.  NOT YET IMPLEMENTED.

* __/v1/analytics/:analytic/queue (GET)__
  * Description: Query the queue name used by the internal Rt 106 messaging system for the specified algorithm.
  * Returns: Queue name in a JSON structure if the algorithm exists, an error code if the algorithm does not exist.

* __/v1/analytics/:analytic/parameters (GET)__
  * Description: Return the input data and parameters required by the algorithm.
  * Returns: A JSON structure with the above if the algorithm exists, an error code if it does not.

* __/v1/analytics/:analytic/results (GET)__
  * Description: Return the expected results that will be returned by the algorithm (not actual results).
  * Returns: A JSON structure with the above if the algorithm exists, an error code if it does not.

* __/v1/analytics/:analytic/results/display (GET)__
  * Description: Return the display format that will be appropriate for the algorithm execution results.
  * Returns: A JSON structure with the above if the algorithm exists, an error code if it does not.

* __/v1/analytics/:analytic/documentation (GET)__
  * Description: Return documentation on the algorithm.  NOT YET IMPLEMENTED.

* __/v1/analytics/:analytic/classification (GET)__
  * Description:  Return the classifiction of the algorithm type, which can be used for filtering for specific types of algorithms to be displayed in the user interface.
  * Returns: A JSON structure with the above if the algorithm exists, an error code if it does not.

### REST API calls related to datastore access
All the datastore API calls below will return an error code if the item being requested does not exist.

These are mostly GET calls.  The few calls that are POST calls are noted as such.

#### Patient-centric API calls

The API calls in the section are detailed for medical imaging, using the DICOM concepts of _studies_ containing _series_ which contain individual images.  However, the API has been structured to allow for the later inclusion of additional patient data types.

* __/v1/datastore/patients__
  * Description: Returns the list of patients.
  * Returns: An array of JSON objects.

* __/v1/datastore/patients/:patient__
  * Description: Returns the types of information available for a given patient.
  * Returns:  A JSON object where each key is a type of data.

* __/v1/datastore/patients/:patient/imaging/studies__
  * Description: Returns the list of studies for the patient.
  * Returns: An array of JSON objects, each describing an imaging study.

* __/v1/datastore/patients/:patient/imaging/studies/:study/types__
  * Description: Returns the types of data included for a given study.
  * Returns: An array of strings containing just "series" in the initial release.

* __/v1/datastore/patients/:patient/imaging/studies/:study/series__
  * Description:  Returns the list of series (as URI paths) for the patient / study.
  * Returns: An array of JSON objects, each describing an imaging series, both primary and algorithm results.

* __/v1/datastore/patients/:patient/imaging/studies/:study/primary__
  * Description:  Returns the list of primary (acquired by device rather than algorithm-generated) for a given patient / study.
  * Returns: Same as for .../series above, but only the primary series.

* __/v1/datastore/series/:series-path/instances__
  * Description:  Returns the list of images (as URI paths) for the given series.
  * Returns: A JSON structure containing both the list of file names and the list of URI paths.

* __/v1/datastore/patients/:patient/results/:pipeline/steps/:execID/imaging/studies/:study/series__
  * Description:  Returns the URI path for uploading an algorithm-generated series.
  * Returns: A JSON object containing the path string.

* __/v1/datastore/series/:series-path/:format__
  * Description: Downloads a series.
  * Returns: A tar archive containing the series or an error code.

* __/v1/datastore/series/:series-path/:format (POST)__
  * Description:  Upload a series.
  * Returns: A JSON structure containing the path to which the series was loaded, or an error code.

* __/v1/datastore/series/:series-path/:format/force (POST)__
  * Description:  Upload a series, overwriting a series that had already been uploaded to that path.
  * Returns: A JSON structure containing the path to which the series was loaded, or an error code.

* __/v1/datastore/annotation/:series-path/types__
  * Description:  Get the types of annotations for a series.
  * Returns: A JSON structure describing the type of annotation (e.g. "DICOM"), or an error code.

* __/v1/datastore/annotation/:series-path/:format__
  * Description:  Download the annotation for a series.
  * Returns: In the first release, an example, hard-coded set of annotations are returned for testing.

##### Version 2 patient-related datastore calls:

* __/v1/datastore/patients/:patient/exams__
  * Description:  Returns the list of exams for the patient.
  * Returns: A JSON object containing the list of exams.

* __/v1/datastore/patients/:patient/exams/:exam/elements__
  * Description:  Returns the list of data elements for the patient and exam.
  * Returns: A JSON object containing the list of data elements.

* __/v1/datastore/patients/:patient/exams/:exam/elements/:element/formats__
  * Description:  Returns the list of data formats available for a patient / exam / data element.
  * Returns: A JSON object containing the list of formats.

* __/v1/datastore/patients/:patient/exams/:exam/elements/:element/formats/dicom/:study/:series__
  * Description:  Get the path to DICOM data for a patient / exam / data element.
  * Returns: A JSON object containing the path.

* __/v1/datastore/patients/:patient/exams/:exam/elements/:element/formats/:format__
  * Description:  Get the path and filename for single-file data for a patient / exam / data element / format.
  * Returns: A JSON object containing the path and filename.

#### REST API calls related to algorithm pipeline results
These REST calls support querying the datastore when there
is a pipeline of algorithms, potentially having multiple
executions and multiple outputs per pipeline step.


* __/v1/datastore/patients/:patient/executions__
  * Description:  Get the list of result execution IDs for an algorithm pipeline.
  * Returns:  A JSON structure containing the list of execution IDs.

* __/v1/datastore/patients/:patient/executions/:execid/analytics__
  * Description:  Get the list of analytics (algorithms) that have been executed for an execution ID.
  * Returns:  A JSON structure containing the list of analytics.

* __/v1/datastore/patients/:patient/executions/:execid/analytics/:analytic/results__
  * Description:  Get the list of result types that have been generated for the execution of the analytic.
  * Returns:  A JSON structure containing the list of results.

* __/v1/datastore/patients/:patient/executions/:execid/analytics/:analytic/results/root__
  * Description:  Get the root path to a result set for an algorithm execution for a patient. Metadata may be stored here.
  * Returns:  A JSON structure containing the path.

* __/v1/datastore/patients/:patient/executions/:execid/analytics/:analytic/results/:result/formats__
  * Description:  Get the formats available for derived data resulting from algorithm execution.
  * Returns:  A JSON structure containing the list of one or more formats.

* __/v1/datastore/patients/:patient/executions/:execid/analytics/:analytic/results/:result/formats/dicom/:study/:series__
  * Description:  Get the path to a derived (algorithm-generated) DICOM data element for a patient.
  * Returns:  A JSON structure containing the path.

* __/v1/datastore/patients/:patient/executions/:execid/analytics/:analytic/results/:result/formats/:format__
  * Description:  Get the path to a derived (algorithm-generated) single-file data element for a patient.
  * Returns:  A JSON structure containing the path and filename.

### REST API calls related to pathology or microscopy

For these calls, the primary object is the _slide_.  For the case of tissue microarray (TMA) experiments, there can be tissue from many patients on the same slide.

* __/v1/datastore/pathology/slides__
  * Description:  Get the list of microscope slides.
  * Returns: An array of slide names.

* __/v1/datastore/pathology/slides/:slide/types__
  * Description:  Get the types of data included in a slide.
  * Returns: An array of strings including just "regions" in the initial release.

* __/v1/datastore/pathology/slides/:slide/regions__
  * Description:  Get the list of regions for a slide.
  * Returns: An array of region names.

* __/v1/datastore/pathology/slides/:slide/regions/:region/types__
  * Description:  Get the types of data included in a region.
  * Returns: An array of strings which will include "channels" and optionally "results".

* __/v1/datastore/pathology/slides/:slide/regions/:region/channels__
  * Description:  Get the list of channels for a given slide and a given region.
  * Returns: An array of channel names.

* __/v1/datastore/pathology/slides/:slide/regions/:region/channels/:channel/types__
  * Description:  Get the types of data for a given slide, region, and channel.
  * Returns: An array of strings for the data types for the slide, region, and channel, e.g. "tiff16".

* __/v1/datastore/pathology/slides/:slide/regions/:region/channels/:channel/image__
  * Description:  Get the image path for a given slide, region, and channel.
  * Returns: An array of URI path strings.  For most situations there should just be one path in the array.

* __/v1/datastore/pathology/slides/:slide/regions/:region/results__
  * Description:  Get the list of "pipelineid" executions for a given slide and region.
  * Returns: An array of named "execution IDs" or result sets.

* __/v1/datastore/pathology/slides/:slide/regions/:region/results/:pipeline/types__
  * Description:  Get the types of results for a given "pipelineid" execution for a given slide and region.
  * Returns: An array of types of things contained in execution pipelines.  Returns just "steps" in the initial release.

* __/v1/datastore/pathology/slides/:slide/regions/:region/results/:pipeline/steps__
  * Description:  Get the list of "execid" steps for a given "pipelineid" execution for a given slide and region.
  * Returns: An array of named "execIDs" or named steps in the execution pipeline.

* __/v1/datastore/pathology/slides/:slide/regions/:region/results/:pipeline/steps/:execid__
  * Description:  Get the format of the results for a given "pipelineid" execution and "execid" step for a given slide and region.
  * Returns: An array of type strings, which may often have just one element, e.g. "tiff16".

* __/v1/datastore/pathology/slides/:slide/regions/:region/results/:pipeline/steps/:execid/data__
  * Description:  Get the URI path of the result for a given "pipelineid" execution and "execid" step for a given slide and region.
  * Returns: The URI path string.

#### API calls related to instances
Instances can be radiology images, pathology images, data files, or any other type of file.

* __/v1/datastore/instance/:instance-path/type__
  * Description:  Get the type of an instance, e.g. "tiff", "DICOM", "csv", etc.
  * Returns: JSON object containing the type string.

* __/v1/datastore/instance/:instance-path/:format__
  * Description:  Download an instance.
  * Returns: The file itself or an error code.

* __/v1/datastore/instance/:instance-path/:format (POST)__
  * Description:  Upload an instance.  If the instance already exists in the data store (even an earlier version), an error is returned.
  * Returns: A JSON object including the path where the instance was uploaded, or an error code.

* __/v1/datastore/instance/:instance-path/:format/force (POST)__
  * Description:  Upload an instance.  If the instance already exists in the datastore, it is overwritten without an error.
  * Returns: A JSON object including the path where the instance was uploaded, or an error code.

### REST API calls related to system health

* __/v1/health/bad (GET)__
  * Description: Return the list of Rt 106 components that have failed their run-time health checks.
  * Returns: An array of Rt 106 component names.

### Misc. REST API calls

* __/v1/dataconvert/csvtojson/v1/pathology/datafile/:slide/:region/:branch/:channel (GET)__
  * Description:  This is a specialized call used by some custom applications to convert CSV files to a JSON structure.
  * Returns: Converted JSON data or an error code.

## AngularJS Services provided by rt106-app
This services can be used within a custom application within its AngularJS controller.

### alarmService
The purpose of this service is to manage alarms and the internal state of Rt 106.

#### alarmService.displayAlert

##### Overview

Display an alarm in a pop-up window.

##### Usage

alarmService.displayAlert( message );

##### Arguments

|Param|Type|Details|
|-----|----|-------|
|message|string|alert message to display|

#### alarmService.scanForHealth

##### Overview

Query Rt 106 for any of its components that have failed its runtime health checks.  Display an alert the first time a component is reported as bad.

##### Usage

alarmService.scanForHealth()

##### Arguments

(none)

### dynamicDisplayService
The purpose of this service to support dynamically-generated result displays for an algorithm execution, based on the JSON metadata defined for that algorithm.  The goal is to enable custom result displays in the user interface without the need to hard code those custom displays.

The capabilities here are just a limited start for the first Rt 106 release.  The paradigm is that the result display will include up to four images arranged in a row or column (for two images) or a square (for four image).

#### dynamicDisplayService.displayInCell

##### Overview

Display the specified image (pathology image or DICOM series) in the appropriate image viewer with a given color & opacity value.

##### Usage

dynamicDisplayService.displayInCell( format, path, displayStructure, column, row, shape, color, opacity, imageProcessing, detections )

##### Arguments

|Param|Type|Details|
|-----|----|-------|
|format|string|"http:" or "tiff16:"|
|path|string|path where image is stored in the datastore|
|displayStructure|JSON|metadata about how to display the image|
|column|integer (0 or 1)|which column to use (see dynamicDisplayService.setDisplayShape)|
|row|integer (0 or 1)|which row to use (see dynamicDisplayService.setDisplayShape)|
|shape|string|(see dynamicDisplayService.setDisplayShape)|
|color|string|e.g. 'rgb(255,255,255)'|
|opacity|number 0 to 1|opacity value|
|imageProcessing|string|If "toBinaryImage" the image will be binarized.  More image processing operations can be added later.
|detections|list of detection points|detections will be displayed on the rendered image


#### dynamicDisplayService.setDisplayShape

##### Overview

Set the result image display to be a single image, a row, a column, or a square.  

##### Usage

dynamicDisplayService.setDisplayShape( shape )

##### Arguments

|Param|Type|Details|
|-----|----|-------|
|shape|string|'1,1' for single image|
| | |'1,2' for two images in a row|
| | |'2,1' for two images in a column|
| | |'2,2' for four images in a square|

#### dynamicDisplayService.displayResult

##### Overview

Display the image results from an algorithm execution.

##### Usage

dynamicDisplayService.displayResult( executionItem, displayStructure, detections )

##### Arguments

|Param|Type|Details|
|-----|----|-------|
|executionItem|JSON|metadata about the algorithm execution|
|displayStructure|JSON|metadata about how to display the image|
|detections|list of detection points|detections will be displayed on the rendered image


### executionService
The purpose of this service is to manage algorithm executions and to maintain the execution list of past executions.

#### executionService.pollExecList

##### Overview
Poll rt106-server's execution data to load rt106-app's execution list.

##### Usage
executionService.pollExecList( execList, ngScope )

##### Arguments
|Param|Type|Details|
|-----|----|-------|
|execList|JSON|the app's current execution list, into which the new results will be merged|
|ngScope|internal AngularJS structure|internal AngularJS structure, just used here to update the scrollbar in the user interface component that displays the execution list.|

#### executionService.initExecution

##### Overview
Initialize the executionService.  Establish the connection between rt106-app and rt106-server, and tell rt106-server to initialize its execution list from past executions stored in the Rt 106 database.

##### Usage
executionService.initExecution()

##### Arguments
(none)

#### executionService.autofillRadiologyParameters

##### Overview
Automatically fill in certain input parameter values from user interface elements, specifically for radiology applications.

* If there is a parameter of type "series" it will be filled in with the path to the currently selected DICOM series.
* If there is a parameter of type "voxelIndex" it will be filled in with the coordinates of a currently-selected probe point.

##### Usage
executionService.autofillRadiologyParameters( selectedParameters, selectedSeries )

##### Arguments
|Param|Type|Details|
|-----|----|-------|
|selectedParameters|JSON|metadata describing the algorithm's required parameters and their values from the user interface|
|selectedSeries|string|path to currently-selected series|

#### executionService.autofillPathologyParameters

##### Overview
Automatically fill in certain input parameter values from user interface elements, specifically for pathology applications.

* If there is a parameter named "slide" it will be filled in with the value of the slide variable.
* If there is a parameter named "region" it will be filled in with the value of the region variable.
* If there is a parameter named "channel" it will be filled in with the value of the channel variable.
* If there is a parameter named "branch" it will be filled in with the value of the selectedPipeline variable.
* If there is a parameter named "force" it will be filled in with the value of forceOverwrite.


##### Usage
executionService.autofillPathologyParameters( selectedParameters, selectedSlide, selectedRegion, selectedChannel, selectedPipeline, forceOverwrite )

##### Arguments

|Param|Type|Details|
|-----|----|-------|
|selectedParameters|JSON|metadata describing the algorithm's required parameters and their values from the user interface|
|selectedSlide|string|currently-selected slide|
|selectedRegion|string|currently-selected region|
|selectedChannel|string|currently-selected channel|
|selectedPipeline|string|currently-selected pipeline|
|forceOverwrite|boolean|whether existing results should be overwritten

#### executionService.requestAlgoRun

##### Overview
Request an algorithm to run.

##### Usage
executionService.requestAlgoRun( selectedParameters, selectedAlgo )

##### Arguments
|Param|Type|Details|
|-----|----|-------|
|selectedParameters|JSON|metadata describing the algorithm's required parameters and their values from the user interface|
|selectedAlgo|JSON|metadata describing the algorithm|

### pathologyService

#### pathologyService.getSlideList

##### Overview
Get the list of slides.

##### Usage
pathologyService.getSlideList()

##### Arguments
(none)

#### pathologyService.getRegionList

##### Overview
Get the list of regions for a slide.

##### Usage
pathologyService.getRegionList( slide )

##### Arguments

|Param|Type|Details|
|-----|----|-------|
|slide|string|name of a slide|

#### pathologyService.getChannelList

##### Overview
Get the list of channels for a region on a slide.

##### Usage
pathologyService.getChannelList( slide, region )

##### Arguments
|Param|Type|Details|
|-----|----|-------|
|slide|string|name of a slide|
|region|string|name of region on the slide|

#### pathologyService.getPrimaryImagePath

##### Overview
Get the path for querying a pathology image.

##### Usage
pathologyService.getPrimaryImagePath( slide, region, channel )


##### Arguments
|Param|Type|Details|
|-----|----|-------|
|slide|string|name of a slide|
|region|string|name of region on the slide|
|channel|string|name of a channel on a region on the slide

## AngularJS Factories provided by rt106-app

### analyticsFactory

#### analyticsFactory.getAnalytics()

##### Overview
Return a JSON structure describing all available algorithms including their classifications, inputs, parameters, expected results, and how result images should be displayed.

A JavaScript promise is returned.

##### Usage
```javascript
analyticsFactory.getAnalytics().then(function(analytics) {
  // save or manipulate the analytics variable
});
```

##### Arguments
(none)

### cohortFactory

#### cohortFactory.getPatients
##### Overview
Get the list of patients.

A JavaScript promise is returned.


##### Usage
```javascript
cohortFactory.getPatients().then(function(patients) {
  // save or manipulate list of patients
})
.catch(function(reason){
  // handle error
});
```

##### Arguments
(none)

#### cohortFactory.getStudies

##### Overview
Get the list of studies for a patient.

A JavaScript promise is returned.

##### Usage
```javascript
cohortFactory.getStudies(patient).then(function(studies) {
  // save or manipulate list of studies
})
.catch(function(reason){
  // handle error
});
```

##### Arguments
|Param|Type|Details|
|-----|----|-------|
|patient|string|name of a patient|

#### cohortFactory.getSeries

##### Overview
Get the list of series for a patient and study.

A JavaScript promise is returned.

##### Usage
```javascript
cohortFactory.getSeries(patient, study).then(function(series) {
  // save or manipulate list of series
})
.catch(function(reason){
  // handle error
});
```

##### Arguments
|Param|Type|Details|
|-----|----|-------|
|patient|string|name of a patient|
|study|string|name of a study for the patient|

______________________________________________________________
