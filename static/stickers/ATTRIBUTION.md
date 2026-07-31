# Sticker assets

The `.webp` files in this directory are derived from [OpenMoji](https://openmoji.org),
the open-source emoji and icon project, and are licensed
**[CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)**.

Source: <https://github.com/hfg-gmuend/openmoji> (`color/618x618` PNGs)

Changes made: resized to 512×512 and converted to WebP with metadata stripped.
As a derivative work these files remain under CC BY-SA 4.0, and the license
requires this attribution to be preserved.

Files are named by the SHA-256 of their own bytes. That is deliberate: Buzz's
Sonar sticker validation requires an asset URL to be HTTPS, carry no port, and
contain the asset's lowercase SHA-256 somewhere in its path. Renaming a file
would break every pack that references it, and the relay verifies the digest
before caching or serving the bytes.

| Shortcode | File |
|---|---|
| (cover) | `783e81096796b928cc3c039567c7af90213bb83468d17c0d771a3fcaa95fa18d.webp` |
| fire | `8394747f3476cd9cf58bb24472dc17ce3f6ea99053ffbc7b3a40df879f6af24f.webp` |
| rocket | `b5c8e5d17d4714aa0c4b2224150060e9abd66715910a1d06a06a95403101fc57.webp` |
| party | `714da3d02d7a039a0f48ae94679f480585ea8507f544480bd55b31aa534a006f.webp` |
| thinking | `1e0ba346016c2d2480053d56978b4b4e1c227d45478bef790647f620308d0de0.webp` |
| eyes | `8198a8f62be158377a75656fa41245880400548dfc23237ab4cb3e1f55c8951d.webp` |
| hundred | `1b3ae5e3692bd3f935ecaa6d1f786b7504326f1ccf64c0d4d78dfd81f9197f29.webp` |
| bug | `158050cd278f6e6fa94a71d26746e4b6737a2c6be9d8e51943859eff5077f58c.webp` |
| coffee | `02c9b6de9cfdf649ff93441bae33bdbf5e4b205b161cd1462a6cd4f6a5656e74.webp` |
| sparkles | `5b2e418376dfb80da03188d9ecc13bf606e4c1a6bb524c310a9e44007037a556.webp` |
