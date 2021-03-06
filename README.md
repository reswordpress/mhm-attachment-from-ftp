# Attachments from FTP
WordPress plugin. Watches a specified folder in your web hosting. If one or more new image files are added to the folder, then a new [Attachment](https://codex.wordpress.org/Attachments) will be created for each of them. 

This plugin works using the WordPress Cron API.

## Instructions
This plugin is in development and **MIGHT CAUSE ERRORS!!!** Don't use it yet unless you're helping me to debug it.

Detailed instructions will be added as development progresses. The first official version of the plugin will be available directly from the WordPress plugin directory as soon as it is released.

As this plugin works with existing WordPress data in the database, and moves files around on the server, make sure that you make a backup of your site and files before using it. Test it with a couple of files before you commit to using it for large numbers of files.

## Installation and activation
* Download the ZIP from this GitHub repository.
* Upload the folder ``mhm-attachment-from-ftp`` to your plugin directory and activate the plugin in the WordPress Admin area.
* Create a direct subfolder in ``wp-content/uploads`` (or your equivalent) which the script will watch.
* Select this folder and the desired author in the plugin settings. (*Settings » Attachments from FTP*.)
* The plugin's function will be run once every hour by default. It works best if you ensure that a cron task is running on your server at least once per hour. [More information about WordPress cron](https://developer.wordpress.org/plugins/cron/).
* If you want to run the check manually, you can use the [WP Crontrol](https://wordpress.org/plugins/wp-crontrol/) plugin. The hook name is ``mhm-attachment-from-ftp/check_folder``.

## Plugin options
* **Source folder**: the folder on the server which will be watched by the cron task for new image files.
* **Post author**: the author in the website to whom new Attachments will be attributed.
* **Number of files to process**: how many images to process in a single batch. (See plugin options for futher details.)
* **Do not overwrite existing titles or descriptions**: if an Attachment already exists for the new file, then don't overwrite the title, the caption or the description when updating with the new file. Default: OFF. (Title and description will be overwritten, even if the new title and description are empty.)

## Notes
* Subfolders of your selected folder and “invisible” files - like ``DS_Store``, ``.`` and ``..`` paths - are automatically ignored.
* The total number of files processed in a single run is limited. (See *Number of files to process* in the plugin options.)
* Uploaded files must contain regular EXIF data relating to capture date.
* The uploaded file may optionally - ideally - contain values for image title and image caption/description. In Adobe Lightroom, this information is edited using the fields *Title* and *Caption* (in the *EXIF and IPTC* view of the *Metadata* panel).
* Older images - i.e. photos taken first - will be processed first.
* A successfully processed file will be moved (not copied) to a destination folder within the regular WordPress uploads structure. The folder is determined from the original date and time when the photo was taken. (``DateTimeOriginal`` in the EXIF data.)
* For example, a photo taken on 16th October 2016 will usually be moved to the folder ``wp-content/uploads/2016/10``.
* If this data is not available in the EXIF, then the file cannot be processed and it will remain in the original folder to which you uploaded it.
* Files whose names contain spaces will be automatically re-named, in order to avoid compatability issues. For example, a file ``2016.10.16 1234.jpg`` will become ``2016.10.16_1234.jpg``. This renaming happens when the file is moved.
* If there is a file with the same (case-sensitive) name in the target directory, then it will be overwritten.
* When the file has been copied to the target directory, the plugin generates new copies of any smaller files - e.g. thumbnails - which are defined in the general [Thumbnail Sizes](https://codex.wordpress.org/Post_Thumbnails#Thumbnail_Sizes) array.
* This plugin doesn't create any additional images of its own.
* If there is already an Attachment which refers to an image in precisely the same target location, then this entry will be updated and no new Attachment will be generated.
* A pre-existing Attachment will be updated with the *Title* and *Caption* of the new image file. Any former, manually-edited caption or title will be overwritten unless the plugin option *Do not overwrite* is selected.

## Detailed process description
Once the plugin is active and the plugin options are set, then the plugin uses WordPress' internal “cron” process to check a folder on your web server once per hour. If there are any images in the folder, then these are destined to become Attachment entries, which you can view in the *Media* section of WordPress admin and use in your Posts or Pages.

So that a file can be correctly processed, the [EXIF data](https://photographylife.com/what-is-exif-data) needs to contain the original capture date and time of the photo. If you're uploading a normal photo, then this information is available. If you edit the photo in an image editing programme like Photoshop or Lightroom, then make sure that the export function doesn't remove the EXIF data. If you've given the photo a title and description in your image editing programme, this will be automatically added to the Attachment entry in WordPress.

If you upload a photo which you've already added, for example after re-editing, with a new title or description, then the previous entry will be updated and the original file will be replaced. WordPress automatically creates smaller versions of each image, for use as a smaller version or as a thumbnail.

## Actions and filters
### Actions
``mhm-attachment-from-ftp/no_files`` fires when there are no files in the folder.
* Arguments: 
    * Source folder (string)

``mhm-attachment-from-ftp/no_file_date`` fires when the selected file has no creation date available in its EXIF data.
* Arguments:
    * File path (string)
    * EXIF data (array)

``mhm-attachment-from-ftp/no_valid_entries`` fires when there are no valid files in the folder.
* Arguments: 
    * Folder path (string)
    * Files (array)

``mhm-attachment-from-ftp/finished`` fires when the process has completely finished.
* Arguments:
    * Files (array)
    * Processed files (array)

``mhm-attachment-from-ftp/source-folder-undefined`` fires when the source folder is not defined.
* Arguments: 
    * Source folder (string)

``mhm-attachment-from-ftp/post-author-undefined`` fires when the post author has not been correctly defined.
* Arguments: 
    * Post author (integer)

``mhm-attachment-from-ftp/source-folder-unavailable`` fires when the indicated source folder is not available on the server.
* Arguments:
    * Folder path (string)

``mhm-attachment-from-ftp/filetype-not-allowed`` fires when a file has a MIME type which is not in the ``allowed_file_types`` array.
* Arguments:
    * File path (string)
    * The MIME type of the file (string)
    * The array of allowed file types. (array)

``mhm-attachment-from-ftp/target_folder_missing`` fires when the target directory, to which a file is to be moved, does not exist.
* Arguments:
    * Folder path (string)

``mhm-attachment-from-ftp/file_moved`` fires after a file is successfully moved.
* Arguments:
    * Source path (string)
    * Target path (string)

``mhm-attachment-from-ftp/file_not_moved`` fires after a file cannot be successfully moved.
* Arguments:
    * Source path (string)
    * Target path (string)

``mhm-attachment-from-ftp/title_description_overwritten`` fires when the title and description of a pre-existing Attachment have been replaced.
* Arguments:
    * Attachment ID (integer)
    * New Attachment data (array)

``mhm-attachment-from-ftp/attachment_updated`` fires when an Attachment has been updated in the database.
* Arguments:
    * Attachment ID (integer)

``mhm-attachment-from-ftp/attachment_created`` fires when an Attachment has been created in the database.
* Arguments:
    * Attachment ID (integer)

``mhm-attachment-from-ftp/updated_attachment_metadata`` fires when the metadata from the Attachment has been updated in the database.
* Arguments:
    * Attachment ID (integer)
    * File path (string)

### Filters
``mhm-attachment-from-ftp/files-in-folder``
* Arguments:
    * Files in the folder (array)

``mhm-attachment-from-ftp/allowed-file-types`` 
* Arguments:
    * Array of allowed file types (array)

## Author
Mark Howells-Mead | www.permanenttourist.ch | Since 11th October 2016

## License
Use this code freely, widely and for free. Provision of this code provides and implies no guarantee.

Please respect the GPL v3 licence, which is available via http://www.gnu.org/licenses/gpl-3.0.html
