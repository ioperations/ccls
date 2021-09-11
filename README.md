## Ccls, which originates from [cquery](https://github.com/cquery-project/cquery), is a C/C++/Objective-C language server. 

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

## improvement
* {hoverProvider completionProvider,codeActionProvider} option in initializationOptions

```json
{
    "initializationOptions": {
        "client": {
            "snippetSupport": true,
            "hoverProvider": false,
            "completionProvider": false,
            "codeActionProvider": false
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

- fix

- from 
```cpp
struct Diagnostic {
    lsRange range;
    int severity = 0;
    int code = 0;
    std::string source = "ccls";
    std::string message;
    std::vector<DiagnosticRelatedInformation> relatedInformation;
    std::vector<TextEdit> fixits_;
};
```

- to 
```
struct Diagnostic {
    lsRange range;
    int severity = 0;
    std::string code = "";
    std::string source = "ccls";
    std::string message;
    std::vector<DiagnosticRelatedInformation> relatedInformation;
    std::vector<TextEdit> fixits_;
};
```

- cause clangd use code as type string , if you use ccls with clangd both as your language server, the codeaction will not work for ccls
- but ccls standalone could be ok !!!

- below is the message sent by coc.nvim ( a vim language client )
```json
{
    "jsonrpc": "2.0",
    "id": 3,
    "method": "textDocument/codeAction",
    "params": {
        "textDocument": {
            "uri": "file:///home/tablinux/scripts/z.cc"
        },
        "range": {
            "start": {
                "line": 117,
                "character": 0
            },
            "end": {
                "line": 118,
                "character": 0
            }
        },
        "context": {
            "diagnostics": [
                {
                    "range": {
                        "start": {
                            "line": 117,
                            "character": 13
                        },
                        "end": {
                            "line": 117,
                            "character": 16
                        }
                    },
                    "severity": 1,
                    "code": 2,
                    "source": "ccls",
                    "message": "no member named 'out' in namespace 'std'; did you mean 'cout'?",
                    "relatedInformation": [
                        {
                            "location": {
                                "uri": "file:///include/c%2B%2B/11.1.0/iostream",
                                "range": {
                                    "start": {
                                        "line": 60,
                                        "character": 17
                                    },
                                    "end": {
                                        "line": 60,
                                        "character": 21
                                    }
                                }
                            },
                            "message": "'cout' declared here"
                        }
                    ]
                },
                {
                    "code": "no_member_suggest",
                    "message": "No member named 'out' in namespace 'std'; did you mean 'cout'? (fix available)",
                    "range": {
                        "end": {
                            "character": 16,
                            "line": 117
                        },
                        "start": {
                            "character": 13,
                            "line": 117
                        }
                    },
                    "relatedInformation": [
                        {
                            "location": {
                                "range": {
                                    "end": {
                                        "character": 21,
                                        "line": 60
                                    },
                                    "start": {
                                        "character": 17,
                                        "line": 60
                                    }
                                },
                                "uri": "file:///usr/include/c%2B%2B/11.1.0/iostream"
                            },
                            "message": "'cout' declared here"
                        }
                    ],
                    "severity": 1,
                    "source": "clang"
                }
            ],
            "only": [
                "quickfix"
            ]
        }
    }
}
```


**ccls can index itself (~180MiB RSS when idle, noted on 2018-09-01), FreeBSD, glibc, Linux, LLVM (~1800MiB RSS), musl (~60MiB RSS), ... with decent memory footprint.**
