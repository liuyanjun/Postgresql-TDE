### todo


Full database encryption support
================================

This version of PostgreSQL is modified with support for full database 
encryption.

Full database encryption feature provides data-at-rest encryption for the 
entire database completely transparently to database clients. The data is 
decrypted when read in from disk and encrypted when written out. Due to this 
the data is not protected from privilege escalation attacks on the database 
server. A misbehaving application can also allow an attacker to gain access 
to data they should not have access to. For use cases that require 
protection from such attacks consider using application side encryption of 
confidential tables/columns. Full database encryption protects against leaks 
of database storage.

Overview
--------

Full database encryption adds an extensibility mechanism for intercepting 
all database reads and writes so encryption can be applied. This mechanism 
allows for any tweakable wide-block encryption to be applied. Key setup 
mechanism is enitrely left to the extension module.

PostgreSQL encrypts all heap files (tables, indexes, sequences, visibility 
and free space maps), including system catalogs, writeahead log, all SLRU's 
(clog, commit_ts, multixact, notify, subtrans) and temporary files for query 
execution.

Pgcrypto contrib module uses this mechanism to implement XTS-AES encryption 
mode with AES-128 cipher. The encryption key could be passed in as an 
environment variable or using an arbitrary shell command. When using the 
environment variable the key is passed through SHA-256 for key setup.

Setting up database encryption
------------------------------

Database encryption can only be enabled at initdb time. Set the desired 
encryption module (a shared library) using --data-encryption parameter. It
is best to also turn on data checksums for some additional security:

    read -sp "Postgres passphrase: " PGENCRYPTIONKEY
    export PGENCRYPTIONKEY
    initdb --data-encryption pgcrypto --data-checksums -D cryptotest

The initdb command will include in postgresql.conf the following 
configuration line:

    encryption_library = 'pgcrypto'

This makes PostgreSQL import pgcrypto in the right position of startup 
procedure and gives it the chance to register itself as an encryption 
provider.

To start the server have PGENCRYPTIONKEY variable with the passphrase 
exported in your environment while you start pg_ctl.

    read -sp "Postgres passphrase: " PGENCRYPTIONKEY
    export PGENCRYPTIONKEY
    pg_ctl -D cryptotest start

Optionally if you want to implement a custom procedure for looking up the 
encryption key using pgcrypto.keysetup_command postgresql.conf parameter. 
When this parameter is present it will be executed by PostgreSQL at startup 
and the output processed. Expected output is a string containing 
encryptionkey= and the 256bit key encoded as a hex string (64 hex 
characters):

    "encryptionkey=" ( [0-9a-f]{64} )

To calculate the key from the passphrase hash it using SHA-256.

Encryption key verification
---------------------------

A known string encrypted using the encryption mechanism is stored in control 
data file. If the encryption module configured by `encryption_library` is not
able to decrypt this string PostgreSQL will refuse to start up logging an 
appropriate error message.

Copyright
---------

Encryption module contains code from Brian Gladman licensed with the 
following terms:

---------------------------------------------------------------------------
Copyright (c) 1998-2008, Brian Gladman, Worcester, UK. All rights reserved.

LICENSE TERMS

The redistribution and use of this software (with or without changes)
is allowed without the payment of fees or royalties provided that:

 1. source code distributions include the above copyright notice, this
    list of conditions and the following disclaimer;

 2. binary distributions include the above copyright notice, this list
    of conditions and the following disclaimer in their documentation;

 3. the name of the copyright holder is not used to endorse products
    built using this software without specific written permission.

DISCLAIMER

This software is provided 'as is' with no explicit or implied warranties
in respect of its properties, including, but not limited to, correctness
and/or fitness for purpose.
---------------------------------------------------------------------------
 Issue Date: 20/12/2007

