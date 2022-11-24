# Test Instructions 

1. Run `dismiss-alerts.yml` workflow to get baseline analysis. This should have 31 open and 2 closed code scanning alerts
2. Add `codeql` comment to the following files:

| File        | Line No.    |
| ----------- | ----------- |
| tests/cpp/main.c |  32    |
| tests/csharp/zipslip.cs | 26 |
| tests/go/main.go | 664 |
| tests/javascript/server.js| 7 |
| tests/python/hash-password.py | 10 |
| tests/ruby/logging.rb | 27 |

3. Run `dismiss-alerts.yml` analysis
