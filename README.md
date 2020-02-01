# Postgresql-TDE
#### Postgresql transparent data encryption is not support officially by PG community. But serveral commercial or open source third party build TDE on Postgresql old version, my target is to build transparent data encryption feature on PG latest version (12.1).
#### I do not want to re-invent the wheel since Cybertech, other company already implement this feature in PG 9.6, I plan to port this implementation into PG last version. (https://www.cybertec-postgresql.com/en/products/postgresql-transparent-data-encryption/)

### <img src="https://www.cybertec-postgresql.com/wp-content/uploads/2017/11/PostgreSQL-instance-level-encryption2.jpg">

# Development Plan
### To-Do
#### 2020.01.26
* TDE概要设计和实现设计
* AES实现算法
* PG TDE 配置
* TDE测试和验证

# High Level Introduction
### Transparent Database Encryption
### <img src="https://github.com/liuyanjun/Postgresql-TDE/blob/master/pg_lowlevel_io.png">

### Database file IO includes DB object, temperary file, log file and database operation
#### Table, Index, Sequence file read and write function
md.c
This code manages relations that reside on magnetic disk.

mdread() -- Read the specified block from a relation.

void mdread(SMgrRelation reln, ForkNumber forknum, BlockNumber blocknum, char *buffer)

mdwrite() -- Write the supplied block at the appropriate location.
 
This is to be used only for updating already-existing blocks of arelation (ie, those before the current EOF).  To extend a relation, use mdextend().
 
void mdwrite(SMgrRelation reln, ForkNumber forknum, BlockNumber blocknum, char *buffer, bool skipFsync)


 mdextend() -- Add a block to the specified relation.
 
 The semantics are nearly the same as mdwrite(): write at the
 specified position.  However, this is to be used for the case of
 extending a relation (i.e., blocknum is at or beyond the current
 EOF).  Note that we assume writing a block beyond current EOF
 causes intervening file space to become filled with zeroes.
 
void mdextend(SMgrRelation reln, ForkNumber forknum, BlockNumber blocknum, char *buffer, bool skipFsync)

#### temperary file read and write function for sorting and indexing
/backend/storage/file/buffile.c
Management of large buffered files, primarily temporary files.

static BufFile *makeBufFile(File firstfile);

static void extendBufFile(BufFile *file);

static void BufFileLoadBuffer(BufFile *file);

static void BufFileDumpBuffer(BufFile *file);

static int	BufFileFlush(BufFile *file);

#### Write ahead log file read and write function
xlog.c
PostgreSQL transaction log manager


  Write and/or fsync the log at least as far as WriteRqst indicates.
 
  If flexible == TRUE, we don't have to write as far as WriteRqst, but
  may stop at any convenient boundary (such as a cache or logfile boundary).
  This option allows us to avoid uselessly issuing multiple writes when a
  single one would do.
 
  Must be called with WALWriteLock held. WaitXLogInsertionsToFinish(WriteRqst)
  must be called before grabbing the lock, to make sure the data is ready to
  write.
 
static void
XLogWrite(XLogwrtRqst WriteRqst, bool flexible)


  I/O routines for pg_control
 
  ControlFile is a buffer in shared memory that holds an image of the
  contents of pg_control.  WriteControlFile() initializes pg_control
  given a preloaded buffer, ReadControlFile() loads the buffer from
  the pg_control file (during postmaster or standalone-backend startup),
  and UpdateControlFile() rewrites pg_control after we modify xlog state.
 
  For simplicity, WriteControlFile() initializes the fields of pg_control
  that are related to checking backend/database compatibility, and
  ReadControlFile() verifies they are correct.  We could split out the
  I/O and compatibility-check functions, but there seems no need currently.
 
static void
WriteControlFile(void)


xlogutils.c

PostgreSQL transaction log manager utility routines




  Read 'count' bytes from WAL into 'buf', starting at location 'startptr'
  in timeline 'tli'.
 
  Will open, and keep open, one WAL segment stored in the static file
  descriptor 'sendFile'. This means if XLogRead is used once, there will
  always be one descriptor left open until the process ends, but never
  more than one.
 
  XXX This is very similar to pg_xlogdump's XLogDumpXLogRead and to XLogRead
  in walsender.c but for small differences (such as lack of elog() in
  frontend).  Probably these should be merged at some point.
 
static void
XLogRead(char *buf, TimeLineID tli, XLogRecPtr startptr, Size count)

#### CLOG, SUBTRANS, MULTIXACT LOG file read and write function
slru.c
Simple LRU buffering for transaction status logfiles


 Physical read of a (previously existing) page into a buffer slot
 
  On failure, we cannot just ereport(ERROR) since caller has put state in
  shared memory that must be undone.  So, we return FALSE and save enough
  info in static variables to let SlruReportIOError make the report.
 
  For now, assume it's not worth keeping a file pointer open across
  read/write operations.  We could cache one virtual file pointer ...
 
static bool
SlruPhysicalReadPage(SlruCtl ctl, int pageno, int slotno)


  Physical write of a page from a buffer slot
 
  On failure, we cannot just ereport(ERROR) since caller has put state in
  shared memory that must be undone.  So, we return FALSE and save enough
  info in static variables to let SlruReportIOError make the report.
 
  For now, assume it's not worth keeping a file pointer open across
  independent read/write operations.  We do batch operations during
  SimpleLruFlush, though.
 
  fdata is NULL for a standalone write, pointer to open-file info during
  SimpleLruFlush.
 
static bool
SlruPhysicalWritePage(SlruCtl ctl, int pageno, int slotno, SlruFlush fdata)

#### create, move, redo database operation
 dbcommands.c
 atabase management commands (create/drop database).
 
DATABASE resource manager's routines
void dbase_redo(XLogReaderState *record)

CREATE DATABASE
Oid createdb(const CreatedbStmt *stmt)

ALTER DATABASE SET TABLESPACE
static void movedb(const char *dbname, const char *tblspcname)

### Advanced Encryption Standard
### <img src="https://raw.githubusercontent.com/oYo-Byte/img_libs/master/blog/165259_mERh_2910723.png">



# Known Issues
#### This is whole database encryption implementation, user has no choice to choose which database is encrypted, which one is not encrypted. this is known limitation.

# References
### Website
* https://www.postgresql.org/
* https://www.cybertec-postgresql.com/
* https://oyo-byte.github.io/2018/10/25/postgresql_fde_encryption/
* http://momjian.us/
* https://billtian.github.io/digoal.blog/
### Books
* 《PostgreSQL数据库内核分析》
* 《PostgreSQL技术内幕》
