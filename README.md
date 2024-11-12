# axfs_ramfs

RAM filesystem implementation.

The implementation is based on `axfs_vfs`.

## Examples

```rust
#![no_std]
#![no_main]

#[macro_use]
extern crate axlog2;
extern crate alloc;
use core::panic::PanicInfo;

mod basic;
mod bench;
mod boundary;

/// Entry
#[no_mangle]
pub extern "Rust" fn runtime_main(_cpu_id: usize, _dtb_pa: usize) {
    axlog2::init("info");
    info!("[rt_ramfs]: ...");

    axalloc::init();

    basic::test_basic();
    bench::test_write();
    boundary::test_boundary().unwrap();

    info!("[rt_ramfs]: ok!");
    axhal::misc::terminate();
}

#[panic_handler]
pub fn panic(info: &PanicInfo) -> ! {
    error!("{}", info);
    arch_boot::panic(info)
}
```

## Struct

### `DirNode`

```rust
pub struct DirNode { /* private fields */ }
```

The directory node in the RAM filesystem.

It implements axfs_vfs::VfsNodeOps.

#### Implementations

##### `impl DirNode`

```rust
pub fn get_entries(&self) -> Vec<String>
```

Returns a string list of all entries in this directory.

```rust
pub fn exist(&self, name: &str) -> bool
```

Checks whether a node with the given name exists in this directory.

```rust
pub fn create_node(
    &self,
    name: &str,
    ty: VfsNodeType,
    uid: u32,
    gid: u32,
    mode: i32
) -> VfsResult<VfsNodeRef>
```

Creates a new node with the given name and type in this directory.

```rust
pub fn fill_node(&self, name: &str, node: VfsNodeRef) -> VfsResult
```

Fill a existed node with the given name into this directory.

```rust
pub fn remove_node(&self, name: &str) -> VfsResult
```

Removes a node by the given name in this directory.

#### Trait Implementations

##### `impl VfsNodeOps for DirNode`

```rust
fn link(&self, path: &str, node: VfsNodeRef) -> VfsResult
```

Create a hardlink with the given path and node Deprecated: use link_child

```rust
fn symlink(
    &self,
    path: &str,
    target: &str,
    uid: u32,
    gid: u32,
    mode: i32
) -> VfsResult
```

Create a symlink with the given path and target

```rust
fn get_ino(&self) -> usize
```

Get inode number

```rust
fn get_attr(&self) -> VfsResult<VfsNodeAttr>
```

Get the attributes of the node.

```rust
fn set_attr(&self, attr: &VfsNodeAttr, valid: &VfsNodeAttrValid) -> VfsResult
```

Set the attributes of the node.

```rust
fn parent(&self) -> Option<VfsNodeRef>
```

Get the parent directory of this directory. Read more

```rust
fn lookup(
    self: Arc<Self>,
    path: &str,
    flags: i32
) -> VfsResult<(VfsNodeRef, String)>
```

Lookup the node with given path in the directory. Read more

```rust
fn read_dir(
    &self,
    start_idx: usize,
    dirents: &mut [VfsDirEntry]
) -> VfsResult<usize>
```

Read directory entries into dirents, starting from start_idx.

```rust
fn create(
    &self,
    path: &str,
    ty: VfsNodeType,
    uid: u32,
    gid: u32,
    mode: i32
) -> VfsResult
```

Create a new node with the given path in the directory Read more

```rust
fn remove(&self, path: &str) -> VfsResult
```

Remove the node with the given path in the directory.

```rust
fn getdents(&self, offset: u64, buf: &mut [u8]) -> VfsResult<usize>
```

Get dir entries from dir node.

```rust
fn write_at(&self, _offset: u64, _buf: &[u8]) -> VfsResult<usize>
```

Write data to the file at the given offset.

```rust
fn fsync(&self) -> VfsResult
```

Flush the file, synchronize the data to disk.

```rust
fn truncate(&self, _size: u64) -> VfsResult
```

Truncate the file to the given size.

```rust
fn as_any(&self) -> &dyn Any
```

Convert &self to &dyn Any that can use Any::downcast_ref.

```rust
fn open(&self, _mode: i32) -> Result<(), AxError>
```

Do something when the node is opened.

```rust
fn release(&self) -> Result<(), AxError>
```

Do something when the node is closed.

```rust
fn read_at(&self, _offset: u64, _buf: &mut [u8]) -> Result<usize, AxError>
```

Read data from the file at the given offset.

```rust
fn link_child(
    &self,
    _fname: &str,
    _node: Arc<dyn VfsNodeOps>
) -> Result<(), AxError>
```

Create a hardlink with the given fname and node Note: Compared with link, fname cannot be a path. So child is a direct child of dir. Read more

```rust
fn create_child(
    &self,
    _fname: &str,
    _ty: VfsNodeType,
    _uid: u32,
    _gid: u32,
    _mode: i32
) -> Result<Arc<dyn VfsNodeOps>, AxError>
```

Create a new node with the given fname in the directory Note: Compared with create, fname cannot be a path. So child is a direct child of dir. Read more

```rust
fn rename(&self, _src_path: &str, _dst_path: &str) -> Result<(), AxError>
```

Renames or moves existing file or directory.

```rust
fn ioctl(&self, _req: usize, _data: usize) -> Result<usize, AxError>
```

Ioctl device.

### `FileNode`

```rust
pub struct FileNode { /* private fields */ }
```

The file node in the RAM filesystem.

It implements axfs_vfs::VfsNodeOps. Content stores pages in btreemap: {page_index => content_in_page}

#### Trait Implementations

##### `impl VfsNodeOps for FileNode`

```rust
fn get_ino(&self) -> usize
```

Get inode number

```rust
fn get_attr(&self) -> VfsResult<VfsNodeAttr>
```

Get the attributes of the node.

```rust
fn set_attr(&self, attr: &VfsNodeAttr, valid: &VfsNodeAttrValid) -> VfsResult
```

Set the attributes of the node.

```rust
fn truncate(&self, size: u64) -> VfsResult
```

Truncate the file to the given size.

```rust
fn read_at(&self, pos: u64, buf: &mut [u8]) -> VfsResult<usize>
```

Read data from the file at the given offset.

```rust
fn write_at(&self, pos: u64, buf: &[u8]) -> VfsResult<usize>
```

Write data to the file at the given offset.

```rust
fn lookup(
    self: Arc<Self>,
    _path: &str,
    _flags: i32
) -> VfsResult<(VfsNodeRef, String)>
```

Lookup the node with given path in the directory. Read more

```rust
fn create(
    &self,
    _path: &str,
    _ty: VfsNodeType,
    _uid: u32,
    _gid: u32,
    _mode: i32
) -> VfsResult
```

Create a new node with the given path in the directory Read more

```rust
fn remove(&self, _path: &str) -> VfsResult
```

Remove the node with the given path in the directory.

```rust
fn read_dir(
    &self,
    _start_idx: usize,
    _dirents: &mut [VfsDirEntry]
) -> VfsResult<usize>
```

Read directory entries into dirents, starting from start_idx.

```rust
fn as_any(&self) -> &dyn Any
```

Convert &self to &dyn Any that can use Any::downcast_ref.

```rust
fn open(&self, _mode: i32) -> Result<(), AxError>
```

Do something when the node is opened.

```rust
fn release(&self) -> Result<(), AxError>
```

Do something when the node is closed.

```rust
fn getdents(&self, _offset: u64, _buf: &mut [u8]) -> Result<usize, AxError>
```

Get dir entries from dir node.

```rust
fn fsync(&self) -> Result<(), AxError>
```

Flush the file, synchronize the data to disk.

```rust
fn parent(&self) -> Option<Arc<dyn VfsNodeOps>>
```

Get the parent directory of this directory. Read more

```rust
fn link(&self, _path: &str, _node: Arc<dyn VfsNodeOps>) -> Result<(), AxError>
```

Create a hardlink with the given path and node Deprecated: use link_child

```rust
fn link_child(
    &self,
    _fname: &str,
    _node: Arc<dyn VfsNodeOps>
) -> Result<(), AxError>
```

Create a hardlink with the given fname and node Note: Compared with link, fname cannot be a path. So child is a direct child of dir. Read more

```rust
fn symlink(
    &self,
    _path: &str,
    _target: &str,
    _uid: u32,
    _gid: u32,
    _mode: i32
) -> Result<(), AxError>
```

Create a symlink with the given path and target

```rust
fn create_child(
    &self,
    _fname: &str,
    _ty: VfsNodeType,
    _uid: u32,
    _gid: u32,
    _mode: i32
) -> Result<Arc<dyn VfsNodeOps>, AxError>
```

Create a new node with the given fname in the directory Note: Compared with create, fname cannot be a path. So child is a direct child of dir. Read more

```rust
fn rename(&self, _src_path: &str, _dst_path: &str) -> Result<(), AxError>
```

Renames or moves existing file or directory.

```rust
fn ioctl(&self, _req: usize, _data: usize) -> Result<usize, AxError>
```

Ioctl device.

### `RamFileSystem`

```rust
pub struct RamFileSystem { /* private fields */ }
```

A RAM filesystem that implements axfs_vfs::VfsOps.

#### Implementations

##### `impl RamFileSystem`

```rust
pub fn new(uid: u32, gid: u32, mode: i32) -> Self
```

Create a new instance.

```rust
pub fn root_dir_node(&self) -> Arc<DirNode>
```

Returns the root directory node in `Arc<DirNode>`.

#### Trait Implementations

##### `impl Default for RamFileSystem`

```rust
fn default() -> Self
```

Returns the “default value” for a type. Read more

##### `impl VfsOps for RamFileSystem`

```rust
fn mount(&self, _path: &str, mount_point: VfsNodeRef) -> VfsResult
```

Do something when the filesystem is mounted.

```rust
fn root_dir(&self) -> VfsNodeRef
```

Get the root directory of the filesystem.

```rust
fn statfs(&self) -> VfsResult<FileSystemInfo>
```

Get the attributes of the filesystem.

```rust
fn alloc_inode(
    &self,
    ty: VfsNodeType,
    uid: u32,
    gid: u32,
    mode: i32
) -> VfsResult<VfsNodeRef>
```

Alloc a new inode.

```rust
fn umount(&self) -> Result<(), AxError>
```

Do something when the filesystem is unmounted.

```rust
fn format(&self) -> Result<(), AxError>
```

Format the filesystem.
