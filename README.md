NOVA Compiler Studio

An interactive compiler-learning platform that compiles a real C subset through a full pipeline and visualizes every phase:

source → lexer → parser → semantic analysis → three-address code → optimizer → bytecode → stack VM
Features
One panel per compiler phase: tokens, AST, symbol table, TAC with optimizer metrics, bytecode, a step-through VM visualizer, and an interactive console with scanf input.
Two interchangeable engines:
Browser engine (src/engine/compilerEngine.js): a complete compiler written in JavaScript; the reference implementation.
Native backend (c_backend/): a faithful C99 mirror exposed over an HTTP API.
Same JSON contract on both engines, enforced by a parity test.
Graceful fallback: if the native server is unreachable, the UI falls back to the browser engine and shows a visible notice.
Tech stack

React 18 · Vite · Tailwind CSS · C99 (native backend) · Node.js (production server)

Quick start

Requires Node.js ^20.19.0 or >=22.12.0.

bash
npm install
npm run dev          # http://localhost:3000 (browser engine by default)

Use the header toggle to switch between In-browser engine and Native C backend. In native mode the UI calls /api/compile, which the Vite dev server proxies to nova_server (default http://127.0.0.1:8080, override with NOVA_BACKEND_URL).

Start the native backend

Requires a C99 toolchain (gcc/clang + make).

bash
npm run backend:build     # builds nova_compiler, nova_server, nova_native_test
npm run backend:server    # serves http://127.0.0.1:8080

The server binds to 127.0.0.1 by default; override with NOVA_HOST / NOVA_PORT.

Use the CLI directly
bash
./c_backend/nova_compiler file.c     # compile a file
cat file.c | ./c_backend/nova_compiler -   # or read from stdin
Tests
Command	What it runs
npm run test:engine	579 assertions on the browser engine (Node, zero deps)
npm run test:c-backend	121 assertions on the native backend (make check)
npm run test:parity	Deep-compares JS vs C JSON across 68 programs
npm run test:gcc	Compares both engines byte-for-byte with GCC across 16 programs
npm run test:stress	Builds the GNU C stress corpus with -Werror and runs all 12 suites
npm test	engine + parity
npm run test:all	engine + native + parity + GCC oracle + stress corpus

Why parity matters: for every program, both engines are compiled and the full result is deep-compared (tokens, AST, symbol table, TAC, optimized TAC, metrics, bytecode, VM trace, console output, diagnostics). The GCC oracle adds an independent check so a bug mirrored in both engines cannot pass on parity alone.

The native backend also builds clean under AddressSanitizer + UBSan:

bash
cd c_backend && make asan-check
Docker

Multi-stage build: compiles the frontend (vite build) and the native backend (make nova_server, statically linked), then serves dist/ with a small Node server that proxies /api/* to nova_server and exposes /api/health.

bash
docker build -t nova-studio .
docker run --rm -p 3000:3000 nova-studio     # http://localhost:3000
Repository layout
src/engine/compilerEngine.js   browser compiler (lexer → … → VM), reference impl
src/engine/presets.js          example programs for the UI
src/components/                phase visualizers, editor, console
c_backend/                     C99 mirror: lexer, parser, semantic, tac, optimizer,
                               bytecode, vm, json, compile + CLI + HTTP server
  main.c                       CLI: nova_compiler file.c (or `-` for stdin)
  server.c                     HTTP API server (nova_server)
  native_test.c                native test harness
tests/run_engine_test.mjs      browser-engine test suite
tests/parity_test.mjs          JS↔C parity suite
tests/corpus/stress_test.c     advanced C stress program (documented limits)
scripts/serve.mjs              production static + proxy server (Docker CMD)
docs/SCHEMA.md                 JSON contract + language subset
AUDIT.md                       audit findings + fix roadmap
Supported language subset (v1)

Supported

Scalars: int, char, float, double
Fixed arrays with initializer lists; structs with member access
Pointers (&, *, pointer parameters)
Control flow: if/else, while, do-while, for, switch/case/default (with fallthrough), goto/labels, break/continue, return
Ternary operator, explicit casts, sizeof(type), compound and bitwise assignments
Functions with recursion
printf / scanf with format strings
Built-ins: sqrt, pow, sin, cos, fabs, rand, srand, assert, exit, and more

Deliberately not supported (each produces a clear diagnostic)

typedef, union, enum, bitfields, function pointers, variadic functions, nested function definitions, the C preprocessor (directives are skipped), and libc/heap/file I/O.

See docs/SCHEMA.md for the JSON contract and full language details.

Further reading
AUDIT.md: audit findings and the fix roadmap this project was rebuilt against
docs/SCHEMA.md: JSON output contract and language subset
Contributing
Make your change in the browser engine and its C mirror (the two must stay in sync).
Run npm run test:all and confirm there are 0 parity mismatches.
For native changes, also run cd c_backend && make asan-check.
License

Add your license here (e.g. MIT) and include a LICENSE file.
