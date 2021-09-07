# ccls

## some improvement
- hoverProvider option in initializationOptions

```json
{
    "initializationOptions": {
        "client": {
            "snippetSupport": true,
            "hoverProvider": false
            }
        },
        "clang": {
            "extraArgs": [
                "-std=c++20",
                "-Wall"
            ]
        }
    }
}
```

ccls, which originates from [cquery](https://github.com/cquery-project/cquery), is a C/C++/Objective-C language server.

  * code completion (with both signature help and snippets)
  * [definition](src/messages/textDocument_definition.cc)/[references](src/messages/textDocument_references.cc), and other cross references
  * cross reference extensions: `$ccls/call` `$ccls/inheritance` `$ccls/member` `$ccls/vars` ...
  * formatting
  * hierarchies: [call (caller/callee) hierarchy](src/messages/ccls_call.cc), [inheritance (base/derived) hierarchy](src/messages/ccls_inheritance.cc), [member hierarchy](src/messages/ccls_member.cc)
  * [symbol rename](src/messages/textDocument_rename.cc)
  * [document symbols](src/messages/textDocument_document.cc) and approximate search of [workspace symbol](src/messages/workspace.cc)
  * [hover information](src/messages/textDocument_hover.cc)
  * diagnostics and code actions (clang FixIts)
  * semantic highlighting and preprocessor skipped regions
  * semantic navigation: `$ccls/navigate`

It has a global view of the code base and support a lot of cross reference features
It starts indexing the whole project (including subprojects if exist) concurrently when you open the first file, while the main thread can serve requests before the indexing is complete.
Saving files will incrementally update the index.


ccls can index itself (~180MiB RSS when idle, noted on 2018-09-01), FreeBSD, glibc, Linux, LLVM (~1800MiB RSS), musl (~60MiB RSS), ... with decent memory footprint. 
