[27/10, 21:47] Uwais Achmad TC²⁴ iTS: | 1   | **`ctx = (g.user["id"], item_id, cipher_name)`**: the user ID is a **number** from SQLite | Your `_context_bytes` only accepts strings, so **every upload will crash with a `TypeError`**. They need `(str(g.user["id"]), str(item_id), cipher_name)`. That's the ctx format you wanted to agree on.                               |
| 2   | **One `item_secret` for all 3 ciphers** (it's made before the loop)                       | The plan says one secret per item × cipher, with independent keys. Your salt and labels still keep the keys different, so it isn't broken, but it doesn't match the plan or the report. Fix: move `new_item_secret()` inside the loop. |
| 3   | **`session.get("kek_bytes")`**: the KEK is read from the Flask session                    | Flask's default session **is the cookie**. The plan says the KEK is never put in the cookie (option B: a server-side cache).                                                                                                           |
| 4   | **A dummy KEK fallback, `b"K" * 32`**                                                     | If the session has no KEK, everything is encrypted with a key anyone can guess. This must be removed before submission.                                                                                                                |
| 5   | **`filename_enc` / `mime_type_enc` store the plaintext name**, only `.encode()`d          | The plan says the filename and MIME type are encrypted. Right now the database shows real filenames.                                                                                                                                   |
| 6   | Minor: `sha256_orig` is stored in plaintext                                               | A plaintext hash lets someone confirm a guess about a file's content. Mention it in the limitations, or encrypt it with the metadata.                                                                                                  |

problem in app.py katanya
[27/10, 21:48] Uwais Achmad TC²⁴ iTS: Points 1 and 2 are about **your interface**, so message person 3 now, for example:

> ctx must be strings: `(str(user_id), str(item_id), cipher_name)`, otherwise labcrypto raises TypeError on upload. Also please call `new_item_secret()` inside the cipher loop, one per cipher like in the plan.

Points 3–5 are for person 2 and person 3 to sort out. Point 3 is the session/KEK cache, which person 2 writes and person 3 checks.
