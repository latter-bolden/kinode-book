# VFS API

Useful helper functions can be found in the [nectar_process_lib](https://github.com/uqbar-dao/process_lib)

The VFS API tries to map over the [std::fs](https://doc.rust-lang.org/std/fs/index.html) calls as directly as possible.

Every request takes a path and a corresponding action.
The paths look like normal relative paths within the folder `your_node_home/vfs`, but they include 2 parts at the start, a `package_id` and a `drive`.

Example path: `/your_package:publisher.nec/pkg/`. This folder is usually filled with files put into the /pkg folder when installing with app_store.

Capabilities are checked on the package_id/drive part of the path, when calling CreateDrive you'll be given "Read" and "Write" caps that you can share with other processes.

Other processes within your package will have access by default.
They can open and modify files and directories within their own package_id.

### Opening/Creating a drive

```rust
let drive_path: String = create_drive(our.package_id(), "drive_name")?;
// you can now prepend this path to any files/directories you're interacting with
```

### Files

#### Open a File

```rust
/// Opens a file at path, if no file at path, creates one if boolean create is true.
let file_path = format!("{}/hello.txt", &drive_path);
let file = open_file(&file_path, true);
```

#### Create a File

```rust
/// Creates a file at path, if file found at path, truncates it to 0.
let file_path = format!("{}/hello.txt", &drive_path);
let file = create(&file_path);
```

#### Read a File

```rust
/// Reads the entire file, from start position.
/// Returns a vector of bytes.
let contents = file.read()?;
```

#### Write a File

```rust
/// Write entire slice as the new file.
/// Truncates anything that existed at path before.
let buffer = b"Hello!";
file.write(&buffer)?;
```

#### Write to File

```rust
/// Write buffer to file at current position, overwriting any existing data.
let buffer = b"World!";
file.write_all(&buffer)?;
```

#### Read at position

```rust
/// Read into buffer from current cursor position
/// Returns the amount of bytes read.
let mut buffer = vec![0; 5];
file.read_at(&buffer)?;
```

#### Set Length

```rust
/// Set file length, if given size > underlying file, fills it with 0s.
file.set_len(42)?;
```

#### Seek to a position

```rust
/// Seek file to position.
/// Returns the new position.
let position = SeekFrom::End(0);
file.seek(&position)?;
```

#### Sync

```rust
/// Syncs path file buffers to disk.
file.sync_all()?;
```

#### Metadata

```rust
/// Metadata of a path, returns file type and length.
let metadata = file.metadata()?;
```

### Directories

#### Open a Directory

```rust
/// Opens or creates a directory at path.
/// If trying to create an existing file, will just give you the path.
let dir_path = format!("{}/my_pics", &drive_path);
let dir = open_dir(&file_path, true);
```

#### Read a Directory

```rust
/// Iterates through children of directory, returning a vector of DirEntries.
/// DirEntries contain the path and file type of each child.
let entries = dir.read()?;
```

#### General path Metadata

```rust
/// Metadata of a path, returns file type and length.
let some_path = format!("{}/test", &drive_path);
let metadata = metadata(&some_path)?;
```

### API

```rust
pub struct VfsRequest {
    /// path is always prepended by package_id, the capabilities of the topmost folder are checked
    /// "/your_package:publisher.nec/drive_folder/another_folder_or_file"
    pub path: String,
    pub action: VfsAction,
}

pub enum VfsAction {
    CreateDrive,
    CreateDir,
    CreateDirAll,
    CreateFile,
    OpenFile { create: bool },
    CloseFile,
    Write,
    WriteAt,
    Append,
    SyncAll,
    Read,
    ReadDir,
    ReadToEnd,
    ReadExact(u64),
    ReadToString,
    Seek { seek_from: SeekFrom },
    RemoveFile,
    RemoveDir,
    RemoveDirAll,
    Rename { new_path: String },
    Metadata,
    AddZip,
    CopyFile { new_path: String },
    Len,
    SetLen(u64),
    Hash,
}

pub enum SeekFrom {
    Start(u64),
    End(i64),
    Current(i64),
}

pub enum FileType {
    File,
    Directory,
    Symlink,
    Other,
}

pub struct FileMetadata {
    pub file_type: FileType,
    pub len: u64,
}

pub struct DirEntry {
    pub path: String,
    pub file_type: FileType,
}

pub enum VfsResponse {
    Ok,
    Err(VfsError),
    Read,
    SeekFrom(u64),
    ReadDir(Vec<DirEntry>),
    ReadToString(String),
    Metadata(FileMetadata),
    Len(u64),
    Hash([u8; 32]),
}

pub enum VfsError {
    NoCap { action: String, path: String },
    BadBytes { action: String, path: String },
    BadRequest { error: String },
    ParseError { error: String, path: String },
    IOError { error: String, path: String },
    CapChannelFail { error: String },
    BadJson { error: String },
    NotFound { path: String },
    CreateDirError { path: String, error: String },
}
```
