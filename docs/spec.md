# WEBTOON EPISODE FILE (.wef)

---

***Revision `1-DRAFT` Errata `-`***

**2026-10-27**

## Table of Contents

1. [Introduction](#1-introduction)
   1. [Conventions](#11-conventions)
      1. [Typography](#111-typography)
      2. [Revisions](#112-revisions)
      3. [Data Structures](#113-data-structures)
2. [File Metadata](#2-file-metadata)
   1. [MIME Type](#21-mime-type)
   2. [File Extensions](#22-file-extensions)
3. [WEF Header](#3-wef-header)
4. [WEF Identifier](#4-wef-identifier)
5. [WEF Image Header Table](#5-wef-image-header-table)
6. [WEF Encryption](#6-wef-encryption)
   1. [WEF Encryption Types](#61-wef-encryption-types)
   2. [WEF Encryption Header](#62-wef-encryption-header)
7. [WEF Metadata Header](#7-wef-metadata-header)

## 1. Introduction

Webtoon episode file is a file format focused on storing a webtoon episode,
a digital comic format originating in South Korea, designed to be read
vertically on a smartphone or a computer. This document refers to it using
a shortened version WEF.

This is not affiliated with [WEBTOON](https://www.webtoons.com) in any way.

### 1.1 Conventions

These are the conventions used throughout this document.

#### 1.1.1 Typography

      The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL
      NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",  "MAY", and
      "OPTIONAL" in this document are to be interpreted as described in
      RFC 2119.

#### 1.1.2 Revisions

Updates to this document are considered either revisions or errata as described below:
- A new revision is made when significant changes are made to the document.
  Which changes the way how the file format is being read or written.
- Errata version is made when a typo is found in the document or a minor change is made to the document that does not
  affect the way how the file format is being read or written.

#### 1.1.3 Data Structures

All data structures are presented in C syntax with the padding implicitly added to ensure that the structure alignment
follows the standard C alignment rules.

All the fields must not be null or 0 unless stated otherwise or is a reserved field.

All offset fields should point to a valid structure unless it is nullable.

The file must be in little endian.

## 2. File Metadata

### 2.1. MIME Type

WEF MIME type should be `application/prs.themonhub.webtoon`

### 2.2. File Extensions

WEF file extension should be the followings
- .wef
- .webtoon
- .wt

## 3. WEF Header

The file must contain this structure at the offset 0.

```c++
typedef struct {
    WefIdentifier wef_ident;
    uint64_t wef_imghdri;
    uint64_t wef_imghdro;
    uint64_t wef_encrypto;
    uint64_t wef_encrypts;
    uint64_t wef_metao;
    uint8_t wef_encryptt;
} WefHeader;
```

### wef_imghdri

This field contains the amount of image header in the image header table.

### wef_imghdro

This field contains the offset to the first entry in the image header table.

### wef_encrypto

This field contains the offset to the Encryption header.

### wef_encrypts

This field contains the offset to the Encryption header size.

### wef_metao

This field contains the offset to the Metadata header.
This field can be `0` if the metadata is not present.

### wef_encryptt

This field contains the DRM type as defined in [`WefEncryptionType`](#61-wef-encryption-types).

## 4. WEF Identifier

```c++
#define WEF_MAGIC_LEN 8
#define WEF_MAGIC {0x7F, "WEBTOON"}

typedef enum {
    WEF_VERSION_UNKNOWN = 0x00,
    WEF_VERSION_1 = 0x01
} WefVersion;

typedef struct {
    char wefi_magic[WEF_MAGIC_LEN];
    uint8_t wefi_version;
    char wefi_reserved[7];
} WefIdentifier;
```

### wef_magic

The magic number must be `0x7F` followed by `WEBTOON` in ASCII.

### wef_version

The version must be the same value as the defined `WEF_VERSION_1`.

## 5. WEF Image Header Table

This table contains information about the images in the WEF file.
Each entry in the table is a `WefImageHeader`.
There will always be at least two entries in the table, being
one or more valid table entries and null table entry.

The data of the file may be encrypted or not, depending on the
value of `wef_encryptt`. See [WEF Encryption Types](#61-wef-encryption-types).

The table is ordered from the top of the comic strip to the bottom from 0 to X.

All images must have the same horizontal resolution, and the
image must be in JPEG XL format[^1].

```c++
typedef struct {
    uint64_t wefih_offset;
    uint64_t wefih_size;
} WefImageHeader;
```

### wefih_offset

This field contains the offset to the data of the file.
If the offset is `0`, this entry is the end of the table (null table entry).

### wefih_size

This field contains the size of the file.

## 6. WEF Encryption

### 6.1 WEF Encryption Types

```c++
typedef enum {
    WEF_ENC_NONE = 0x00,
    WEF_ENC_PWD_ARGON2ID_AES_GCM = 0x01
} WefEncryptionType;
```

#### WEF_ENC_NONE

No encryption has been done.

#### WEF_ENC_PWD_ARGON2ID_AES_GCM

The file has been encrypted with a password using Argon2id and AES-GCM.
Every encrypted data must have an AES-GCM tag at the end.
See [WefEncryptionPwdArgon2idAesGcm](#wef_enc_pwd_argon2id_aes_gcm-1).

### 6.2 WEF Encryption Header

This structure contains information about the encryption of the file.
The structure varies depending on the encryption type.

#### WEF_ENC_NONE

Size: 8

```c++
typedef struct {
    uint64_t wefeh_reserved;
} WefEncryptionNoneHeader;
```

#### WEF_ENC_PWD_ARGON2ID_AES_GCM

Size: 44

```c++
#define WEF_ARGON2_SALT_LEN 16
#define WEF_AES_GCM_NONCE_LEN 12
#define WEF_AES_GCM_TAG_LEN 16

typedef struct {
    uint32_t wefeh_tcost;
    uint32_t wefeh_mcost;
    uint32_t wefeh_paral;
    uint8_t wefeh_salt[WEF_ARGON2_SALT_LEN];
    uint8_t wefeh_nonce[WEF_AES_GCM_NONCE_LEN];
    char wefeh_reserved[4];
} WefEncryptionPwdArgon2idAesGcm;
```

##### wefeh_tcost

The number of iterations or time cost of the Argon2id algorithm.

##### wefeh_mcost

Memory cost in KiB of the Argon2id algorithm.

##### wefeh_paral

How many parallel threads to use for the Argon2id algorithm.

##### wefeh_salt

Salt used by Argon2id algorithm.

##### wefeh_nonce

Nonce used by AES-GCM algorithm.

## 7. WEF Metadata Header

This header contains metadata about the WEF file.

```c++
#define WEF_ID_LEN 32

typedef struct {
    char wefmh_id[WEF_ID_LEN];
    uint64_t wefmh_thmno;
    uint64_t wefmh_thmnvo;
    uint64_t wefmh_nameo;
    uint64_t wefmh_names;
    uint64_t wefmh_desco;
    uint64_t wefmh_descs;
} WefMetadataHeader;
```

> [!NOTE]
> `wefmh_thmno` and `wefmh_thmnvo`'s `WefImageHeader` does not appear in the WEF Image Header Table.

### wefmh_id

Identifies the comic this file belongs to.
The identifier is a 32-byte string that can
be null-terminated before hitting the length limit.

The content of this field must be in ASCII,
contain no space, printable characters.

This field should be the same in each episode of the same comic.

### wefmh_thmno

Offset to [`WefImageHeader`](#5-wef-image-header-table)
that contains the comic thumbnail in `1:1` ratio.

### wefmh_thmno

Offset to [`WefImageHeader`](#5-wef-image-header-table)
that contains the thumbnail in `9:16` ratio also known as vertical thumbnail.

### wefmh_nameo

Offset to UTC-8 string that contains the name of the comic.

### wefmh_names

Size of the name string in bytes.

### wefmh_desco

Offset to UTC-8 string that contains the description of the comic.

### wefmh_descs

Size of the description string in bytes.

## Changes

- `1.0` - Initial version.

## Contribution

You can contribute to this document by submitting pull requests on GitHub.
Expansion to the DRM type is welcome and encouraged.

## Contact

In case of any questions or suggestions, please feel free to open an issue on GitHub.

## Copyright

Copyright © 2026 TheMonHub

This work is licensed under a
[Creative Commons Attribution 4.0 International](https://creativecommons.org/licenses/by/4.0/) License.

![CC](https://mirrors.creativecommons.org/presskit/icons/cc.png)
![BY](https://mirrors.creativecommons.org/presskit/icons/by.png)

[^1]: **JPEG XL, JPEG committee** – https://jpegxl.info/