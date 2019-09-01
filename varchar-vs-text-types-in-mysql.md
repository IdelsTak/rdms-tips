# [Difference between VARCHAR and TEXT in mysql](https://stackoverflow.com/questions/25300821/difference-between-varchar-and-text-in-mysql)

**TEXT**

- fixed max size of 65535 characters (you cannot limit the max size)
- takes 2 + `c` bytes of disk space, where `c` is the length of the stored string.
- cannot be part of an index

**VARCHAR(M)**

- variable max size of `M` characters
- `M` needs to be between 1 and 65535
- takes 1 + `c` bytes (for `M` ≤ 255) or 2 + `c` (for 256 ≤ `M` ≤ 65535) bytes of disk space where `c` is the length of the stored string
- can be part of an index

## More Details

`TEXT` has a **fixed** max size of `2¹⁶-1 = 65535` characters.
 `VARCHAR` has a **variable** max size `M` *up to* `M = 2¹⁶-1`.
 So you cannot choose the size of `TEXT` but you can for a `VARCHAR`.

The other difference is, that you cannot put an index (except for a fulltext index) on a `TEXT` column.
 So if you want to have an index on the column, you have to use `VARCHAR`. But notice that the length of an index is also limited, so if your `VARCHAR` column is too long you have to use only the first few characters of the `VARCHAR` column in your index (See the documentation for [`CREATE INDEX`](https://dev.mysql.com/doc/refman/5.7/en/create-index.html)).

But you also want to use `VARCHAR`, if you know that the maximum length of the possible input string is only `M`, e.g. a phone number or a name or something like this. Then you can use `VARCHAR(30)` instead of `TINYTEXT` or `TEXT`  and if someone tries to save the text of all three "Lord of the Ring"  books in your phone number column you only store the first 30 characters  :)

**Edit:** If the text you want to store in the database is longer than 65535 characters, you have to choose `MEDIUMTEXT` or `LONGTEXT`, but be careful: `MEDIUMTEXT` stores strings up to 16 MB, `LONGTEXT` up to 4 GB. If you use `LONGTEXT` and get the data via PHP (at least if you use `mysqli` without `store_result`),  you maybe get a memory allocation error, because PHP tries to allocate 4  GB of memory to be sure the whole string can be buffered. This maybe  also happens in other languages than PHP.

However, you should **always** check the input (Is it too long? Does it contain strange code?) *before* storing it in the database.

Notice: For both types, the required disk space depends only on the length of the stored string and not on the maximum length.
 *E.g.* if you use the charset *latin1* and store the text "Test" in `VARCHAR(30)`, `VARCHAR(100)` and `TINYTEXT`,  it always requires 5 bytes (1 byte to store the length of the string  and 1 byte for each character). If you store the same text in a `VARCHAR(2000)` or a `TEXT`  column, it would also require the same space, but, in this case, it  would be 6 bytes (2 bytes to store the string length and 1 byte for each  character).

For more information have a look at the [documentation](https://dev.mysql.com/doc/refman/5.7/en/blob.html).

Finally, I want to add a notice, that both, `TEXT` and `VARCHAR`  are variable length data types, and so they most likely minimize the  space you need to store the data. But this comes with a trade-off for  performance. If you need better performance, you have to use a fixed  length type like `CHAR`. You can read more about this [here](https://dba.stackexchange.com/a/2643/46158).