`OpusCustomFields` lets you define and set custom fields accessible in Directory Opus for virtually any file, with support for a variety of data types. It's the first read/write script add-in for Directory Opus.

> BEFORE CONTINUING, please keep the following in mind:

- This is a beta version and may contain bugs.
- The whole point of this is for you to test it in real-world use and report any bugs and suggestions.

### Features

* Almost any item can have custom fields, including FTP, MTP, items inside archives, etc.
* Blazing fast!
* You can create fields of any data type supported by Opus. It also supports multivalue data types, with special support for searching them (arrays and sets).
* Manage custom fields through an easy-to-use interface.
* Support for exporting/importing custom fields.
* Support for advanced actions: PULL, PUSH, FLUSH, etc.
* Seamless data management using an SQLite database, with commands for database maintenance.
* Support for directly searching the database, including an advanced mode using SQL SELECT syntax.

And more!

<img width="1346" height="748" alt="config" src="https://github.com/user-attachments/assets/80c578bb-1cff-4ab3-b4d8-0bda09d73d4f" />


### **Requirements**

* Directory Opus 13.25.4 or higher.
* Windows 10 1803 or higher (since the script uses Unix sockets for faster communication with the helper).

---

### **Installation**

[Download the installer from here](https://github.com/departure05/OpusCustomFields/releases/download/latest/OpusCustomFieldsInstaller.exe)

Then run the installer. This will put the helper in place and launch the standard script installer.

> This script uses a Go executable as a helper, which is why it requires a special installer.

---

### **How it works**

The script stores all its data in a database in its configuration folder. It communicates with the database transparently through a helper to manage the data with zero delay.

> Data is stored unencrypted, so avoid storing sensitive information in it.

To make it easier to manage and locate registered files, their paths are stored using the disk's volume name instead of its drive letter, where applicable. This is particularly useful for files on removable or USB drives, which tend to change drive letters.

> This script cannot track files. If a file is moved, renamed, or deleted, its data may be lost.

To work around this limitation, the script uses several methods:

1. For files on NTFS drives, it can back up data to an Alternate Data Stream (ADS). If the file is moved or renamed, the next time you try to retrieve its data, the script can recover it from this source and automatically update the database. (Optional; must be enabled in Options.)
2. For files on NTFS drives, it also stores each item's unique file identifier, allowing an item to be relocated after being moved, provided its identifier hasn't changed.
3. For all files, the script can retrieve data from a sidecar file stored alongside the original file, using the same filename plus the `.dopcf` extension. (Optional; must be enabled in Options.)

> Automatic recovery methods are only used when the file in question has no existing entry in the database.

> If ADS support is enabled, the ADS is automatically updated after every change, so manual editing is not recommended.

> Sidecar files are never generated automatically, but they can be generated manually using the `PULL` command.

> For the ADS workaround to work, you must set `Copy Alternate Data Streams` to `Copy to new file` under `Preferences / File Operations / Copying Files / Metadata (Copying Files)`.

---

### **Configuring fields**

The first time you install the script, you'll be asked if you want to start configuring fields.

To configure which fields are enabled, open the configuration dialog in one of the following ways:

* From the Script Management window, click the Configure button.
* Run the `OpusCustomFields CONFIG` command.

<img width="1346" height="748" alt="config" src="https://github.com/user-attachments/assets/80c578bb-1cff-4ab3-b4d8-0bda09d73d4f" />

Checked fields will be enabled as columns in Opus once you save your changes and close the dialog.

* The **Category** control lets you choose where the column appears. By default, columns are added under Script > OpusCustomFields. You can change this to any category you prefer. This doesn't affect the column's behavior or output.
* `Blur in secure screenshot` lets you specify which columns will be blurred when Secure Screenshot is active.
* You can export or import columns from this window using the provided buttons.
* You can also set a short description for each field (max. 255 characters).

Press the `Add` button to create a new field by entering a name and choosing a type.

* The **Type** control sets the data type of the column's content. Allowed types are: String (text), number (int), double, star (rating), datetime, date, time, duration, size, percentage, array, and set.
> For compatibility reasons, field names must be lowercase, cannot contain spaces, commas, or slashes, and cannot begin with `_`.

> Once a field's name and type have been set, they cannot be changed.

When selecting an existing field, press the `Delete` button to delete it.

> Deleting a field is a destructive operation. All data associated with that field will be deleted from the database, so use this feature with extreme caution.

Once enabled, fields can be accessed and used as regular Opus columns anywhere columns are available.

> There are two internal columns that are always enabled: `_count` (Number of fields) and `_last_update` (Last Saved Date).

---

### **Options**

You can also access the Options dialog from the Config window to configure settings such as the script log level, backup-related options, and more.

(This dialog is also accessible from the Script Management window by clicking the About button.)

<img width="905" height="444" alt="options" src="https://github.com/user-attachments/assets/e7312de9-50d9-42c2-9619-332a9412e296" />

* **Enable ADS support as a recovery resource**: When this option is enabled, if a file has no entry in the database, its ADS can be used as a recovery source. When data is set, the ADS is also updated automatically. Only applies to files on NTFS drives.

* **Enable sidecar files support as a recovery resource (read only)**: When this option is enabled, data can also be recovered from a sidecar file stored alongside the original file, using the same filename plus the `.dopcf` extension. This is for read-only recovery purposes; these files will not be created automatically.

---

### **Command arguments**

| Argument | Type | Value | Description |
| --- | --- | --- | --- |
| **CONFIG** | /S | | Displays the field configuration dialog. |
| **EXPORT** | /O/K | | Exports fields to a file. Also accepts an output path. |
| **FILES** | /M | | Files to act upon. If no files are provided, the script attempts to use the selected files in the source file display. Can be used with `SET`, `FLUSH`, `PULL`, `PUSH`, and `SETUI`. When used with `PURGEDB`, it allows the selected files to be deleted from the database. |
| **FLUSH** | /O/K | | A comma-separated list of field names whose values should be flushed from the database. If omitted, all active fields will be flushed. If ADS support is enabled, their values will be flushed from the ADS as well. |
| **PULL** | /O/K | | Pulls all values from active fields into a `.dopcf` file in JSON format. You can also specify an output filename. |
| **PUSH** | /K | | Pushes values for active fields from a file. The file must contain a JSON object in the `field:value` format. |
| **SET** | /M | | Sets values for active fields. You can pass values for multiple fields, just like with `SET META`. |
| **FIND** | /O/R | | Searches the database. You can pass a raw query or enter one in the FIND dialog. |
| **IN** | /K | | Specifies a path for `FIND` to search within. Must be a valid filesystem-only path. |
| **ADVANCED** | /S | | Enables special SQL syntax for the `FIND` command. |
| **PURGEDB** | /O | | Performs database cleanup, if applicable. |
| | | *path/wildcard* | If a text value is specified, it can be the full name of a specific file or a path containing `*` and `?` wildcards. If omitted, all entries for non-existing items are removed (`KEEPMISSINGDRIVES` can change this behavior). |
| **KEEPMISSINGDRIVES** | /S | | Used with `PURGEDB` without a specified value, this keeps entries located on currently unavailable drives, such as external disks. |
| **VACUUM** | /S | | Compacts the database. Use together with `PURGEDB`. |
| **SETUI** | /O/K | | Sets values through a user interface. |
| | | `autosave` | Automatically saves changes whenever the file selection changes. |
| | | `text` | Starts with all field controls set to accept text. Useful for referencing properties in non-text fields. |
| **NOONTOP** | /S | | Prevents the dialog from opening on top. Can be used with `CONFIG` and `SETUI`. |
| **NOEDIT** | /S | | When used with `SET`, disables the add/remove feature for array-based fields. |

#### Setting values (via `SET`, `PUSH`, or `SETUI`)

Valid values vary depending on the data type of each field.

* **For arrays/sets:**
  - You can pass a comma- or semicolon-separated string (or an actual array when using `PUSH`).
  - A single quote (`'`) can be used to escape a comma if needed.
  - To use placeholders in arrays/sets, you must pass the value as a string.
  - By default, arrays and sets support adding/removing values when used with `SET`, by appending the `+` or `-` suffix to each value.

    `OpusCustomFields SET "tags:+test;-low"` will append `test` and remove `low` (if present).

    - You cannot mix values with and without suffixes. All values must have a suffix when adding/removing items; otherwise, the entire input will be treated as a raw value. For example, `+test;-low;other value` is invalid for add/remove operations and will instead replace the current value with the input as is.

* **For duration values, you can pass:**
  - A number indicating seconds.
  - A string in the format `xYxMxDxHx(m|min)xs`, where `x` represents an integer. Letters are case-insensitive except for `M` and `m`.

* **For size values, you can pass:**
  - A number indicating bytes.
  - A string in any format that Opus recognizes as a valid size, e.g. `20.4MB`.

* **For date-related values, you can pass:**
  - The text `now`.
  - A valid ISO 8601 string in the format `YYYY-MM-ddTHH:mm:ss[.ms]`, or the relevant portion for `date` and `time`. Include `Z` to indicate UTC, or specify a time zone.

To remove a field's value for a file, set it to `""` (empty), regardless of its data type.

You can also reference properties of the files themselves. These can be of any type (script, evaluator, etc.). To reference a property, enclose its internal name in `$_..._$`. For example, `$_created_$` retrieves the creation date.

> Avoid referencing properties known to take a long time to calculate, such as checksums for large files, as this will stall processing for that file and all subsequent files.

* With `SET`, you can pass values using the following format:

  `OpusCustomFields [FILES "...." "..."] SET "field1:value" "field2:value" ...`

  - If `FILES` is omitted, the selected files in the source file display will be used.
  - Values are specified as `field:value` pairs. Use quotes if they contain spaces.

  `OpusCustomFields SET "mydesc:Some definition" "timestamp:$_created_$" myrating`

* `PUSH` accepts a file containing valid JSON in the same `field:value` format. The same value types accepted by `SET` are supported.

  `OpusCustomFields PUSH="c:\path to\my file.json"`

* `SETUI` provides an interface for setting values more dynamically. Since it retrieves each field's current value when opened, it's ideal for editing or adding to existing values.

<img width="1405" height="734" alt="setui" src="https://github.com/user-attachments/assets/fddc6039-83cd-41f6-aa38-32df946a1848" />


  - Use the `Use item's properties` checkbox to switch the controls to text input. This is useful when referencing properties in fields that don't accept text directly.
  - After each file selection change, you must save or discard your changes using the Save button or Ctrl + S. You can enable automatic saving from the Save dropdown or by using `SETUI=autosave`.

---

### FIND syntax

* **SIMPLE mode** is a small expression language parsed in situ and converted into parameterized SQL. The client's text never reaches SQLite.

  ```
  expr      := and ( OR and )*
  and       := primary ( AND primary )*            (AND binds tighter than OR)
  primary   := ( expr ) | NOT ( expr ) | condition
  condition := field op value
             | field IS [NOT] EMPTY
             | field [NOT] IN ( value, ... )
             | field [NOT] BETWEEN value AND value
             | field [NOT] CONTAINS|STARTSWITH|ENDSWITH "text"     (text fields)
             | field [NOT] CONTAINS value                         (array fields - see below)
  op        :==  !=  <>  <  <=  >  >=
  value     := 12  -3.5  true  false  "text"  'text'    (dates go in quotes)
  ```

  - **Missing values:** Comparisons follow SQL semantics, so a file with no value for a field never satisfies `=`, `!=`, `<`, `>`, etc. Use `field IS EMPTY` or `field IS NOT EMPTY` to test for missing values, regardless of field type. `== ""` and `!= ""` are shorthand for `IS EMPTY` and `IS NOT EMPTY`, respectively, for every field type, not just text.

  - **Array fields:** Only `CONTAINS`/`NOT CONTAINS` (checks whether the JSON array contains a specific element, using a plain value in the same format as above, e.g. `tags CONTAINS "rock"` or `scores CONTAINS 5`) and `IS [NOT] EMPTY` (null or `[]`) are supported. `=`, `!=`, `<`, `<=`, `>`, `>=`, `IN`, and `BETWEEN` are not supported for array fields.

  - **Dates:** Accepted formats include:
    - Full date and time, with either `T` or a space between the date and time, with an explicit `Z`/offset for UTC or no offset for local time.
    - Date only (`2026-09-21`) or time only (`15:30:00`).
    - A placeholder keyword: `today`, `yesterday`, `thisweek`, `lastweek`, `thismonth`, `lastmonth`, `thisyear`, `lastyear`.
    - A relative duration directly appended to a number: `7d`, `12h`, `30min`, `2y`, `1.5h`, `45s`. The comparison operator determines what "before" or "after" means. For example, `taken >= 7d` finds anything from the last week onward.

* **ADVANCED mode** wraps the query as follows:

```sql
SELECT _file FROM entries WHERE ( <QUERY> ) [AND _file LIKE <IN>]
```

Here, `<QUERY>` is your actual query, and `<IN>` is the value passed through the `IN` argument, if used.

  - This mode does NOT try to understand SQL. SQLite parses the expression when it runs and reports its own syntax errors. The script only lexes the text to handle strings, quoted names, and comments correctly, rejecting an empty expression and a bare `;` (which would either break out of the `WHERE` clause into a second statement or be a pointless no-op; either way, a single expression should never need one). Otherwise, any expression-level SQL is allowed, including subqueries, `CASE`, `json1` functions, and so on.

* `FIND` has its own dialog where you can enter queries and switch between modes.

<img width="798" height="588" alt="find" src="https://github.com/user-attachments/assets/490d04f4-6de8-4dd9-b486-fe97ba1eb8ef" />

* When searching, you can access the internal columns `_count` (Number of fields) and `_last_update` (Last Saved Date).
* You can access a list of your active fields using the button or F6.
* You can also access your recent search history using the button or F4.

---

### **CLEANING THE DATABASE**

<img width="694" height="659" alt="purgedb" src="https://github.com/user-attachments/assets/f2846359-9248-4d45-8882-4bf2ed0c16b6" />


You can maintain your database using the `PURGEDB` argument.

When used together with a list of `FILES`, this deletes those files from the database, along with their ADS entries, if any.

If you pass a wildcard to `PURGEDB`, or omit `FILES`, the script checks for orphaned entries. You can use `KEEPMISSINGDRIVES` to decide whether to keep entries from removable drives that are not currently available.

A list of files will then be displayed, allowing you to choose which entries to keep and which to delete. After you press OK, the selected entries will be deleted from the database.

> Due to limitations in the Opus scripting API, all files on MTP devices will be listed, even if they exist.

You can also include `VACUUM` to compact the database.

---

### **NOTES**

* TEXT fields have a maximum length of 1,000 characters. Any excess will be automatically truncated.
* When the database reaches 1 GB in size, certain features, such as `FIND`, will become unavailable.
* Fields are NOT indexed, to maintain decent performance when setting values or rearranging the database. Even so, searches should be fast enough.
* This script is NOT DESIGNED for indexing purposes, such as indexing file properties or content.
* Files from remote locations stored in the database may slow down some operations, such as `FIND` or `PURGEDB`.
* ADS reading/writing and sidecar file lookup are effectively supported only for files that report a valid drive and an NTFS filesystem through the Opus scripting API.
* Some operations cannot run simultaneously.
* At least for now, FTP support is disabled.

---

### **Acknowledgments**

* To the Opus team, as usual.

The Go helper uses the following third-party libraries:

- `github.com/goccy/go-json`
- `modernc.org/sqlite`
- `github.com/dustin/go-humanize`
- `github.com/google/uuid`
- `github.com/mattn/go-isatty`
- `github.com/ncruces/go-strftime`
- `github.com/remyoudompheng/bigfft`
- `golang.org/x/sys`
- `modernc.org/libc`
- `modernc.org/mathutil`
- `modernc.org/memory`

---

### **Changelog**

**v0.9.7b (Oct 15, 2026):**

* Initial public release.
