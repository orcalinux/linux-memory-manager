@page dev_guide Developer Guide

@tableofcontents

@section dev_intro Introduction for Developers

This guide is for developers who want to **understand**, **modify**, or **contribute** to the Linux Memory Manager project. If you just want to use the library, see the @ref user_guide instead.

---

@section dev_setup Development Setup

@subsection dev_requirements Requirements

- **GCC 7.0+** - C compiler with C11 support
- **GNU Make** - Build automation
- **Git** - Version control
- **Doxygen 1.9+** - Documentation generation (optional)
- **Graphviz** - Diagram generation (optional)
- **Valgrind** - Memory leak detection (optional)

@subsection dev_clone Clone Repository

@code{.sh}
git clone https://github.com/orcalinux/linux-memory-manager.git
cd linux-memory-manager
@endcode

@subsection dev_build Build Everything

@code{.sh}

# Build all targets

make all

# Or build individually

make static # Build static library only
make shared # Build shared library only
@endcode

---

@section dev_architecture Architecture Overview

@subsection arch_components Core Components

The memory manager consists of several key components:

1. **VM Page Manager** (memory_manager.c)

   - Manages virtual memory pages from the kernel
   - Implements page families for different data types
   - Handles page allocation and deallocation

2. **Block Allocator** (memory_manager.c)

   - Divides pages into fixed-size blocks
   - Maintains free block lists
   - Implements block splitting and merging

3. **GLThread** (glthread.c)

   - Generic intrusive linked list
   - Thread-safe operations
   - Zero-overhead abstraction

4. **Type Registry** (datatype_size_lookup.c)
   - Maps type names to sizes
   - Enables dynamic type registration
   - Used by XCALLOC/XFREE macros

@subsection arch_memory_model Memory Model

@code
┌─────────────────────────────────────────────┐
│ Kernel Virtual Memory │
└─────────────────────────────────────────────┘
↓
┌──────────────────┐
│ VM Page (4KB) │
└──────────────────┘
↓
┌────────────────────────────────┐
│ Block Meta Data | Data Block │
│ Block Meta Data | Data Block │
│ Block Meta Data | Data Block │
│ ... │
└────────────────────────────────┘
@endcode

---

@section dev_code_structure Code Organization

@subsection structure_files File Organization

@code
src/
├── memory_manager.c # Core memory management
├── glthread.c # Linked list implementation
├── datatype_size_lookup.c # Type size registry
├── parse_datatype.c # Type name parsing
└── memory_manager_test.c # Test suite

include/
├── memory_manager_api.h # Public API (users include this)
├── memory_manager.h # Internal structures
├── glthread.h # GLThread API
├── datatype_size_lookup.h # Type registry API
├── parse_datatype.h # Parser API
└── colors.h # Terminal output utilities
@endcode

@subsection structure_headers Header Dependencies

@code
memory_manager_api.h (PUBLIC API)
↓
memory_manager.h (INTERNAL)
↓
glthread.h (DATA STRUCTURES)
@endcode

---

@section dev_building Building & Testing

@subsection dev_make_targets Makefile Targets

| Target           | Description                      |
| ---------------- | -------------------------------- |
| `make all`       | Build executable and libraries   |
| `make static`    | Build static library (libhmm.a)  |
| `make shared`    | Build shared library (libhmm.so) |
| `make clean_all` | Remove all build artifacts       |
| `make clean_obj` | Remove object files only         |
| `make doc`       | Generate Doxygen documentation   |
| `make clean_doc` | Remove generated documentation   |

@subsection dev_testing Testing

**Run the test suite:**
@code{.sh}
./bin/hmm
@endcode

**Run with Valgrind (detect memory leaks):**
@code{.sh}
valgrind --leak-check=full --show-leak-kinds=all ./bin/hmm
@endcode

**Run with AddressSanitizer:**
@code{.sh}
gcc -fsanitize=address -g src/\*.c -o bin/hmm_asan
./bin/hmm_asan
@endcode

---

@section dev_contributing Contributing

@subsection contrib_workflow Contribution Workflow

1. **Fork the repository** on GitHub
2. **Create a feature branch:**
   @code{.sh}
   git checkout -b feature/my-new-feature
   @endcode

3. **Make your changes** and commit:
   @code{.sh}
   git add .
   git commit -m "Add new feature: description"
   @endcode

4. **Push to your fork:**
   @code{.sh}
   git push origin feature/my-new-feature
   @endcode

5. **Open a Pull Request** on GitHub

@subsection contrib_guidelines Coding Guidelines

**Style Rules:**

- Use 4 spaces for indentation (no tabs)
- Maximum line length: 80 characters
- Use snake_case for functions and variables
- Use UPPER_CASE for macros and constants
- Add Doxygen comments for all public functions

**Example:**
@code{.c}
/\*\*

- @brief Allocate a block of memory
-
- @param size Size in bytes to allocate
- @return Pointer to allocated memory, or NULL on failure
  _/
  void_ mm_allocate(size_t size) {
  // Implementation
  }
  @endcode

@subsection contrib_commit Commit Messages

Follow this format:
@code
<type>: <subject>

<body>

<footer>
@endcode

**Types:**

- `feat:` New feature
- `fix:` Bug fix
- `docs:` Documentation changes
- `refactor:` Code refactoring
- `test:` Adding tests
- `chore:` Maintenance tasks

**Example:**
@code
feat: Add memory fragmentation analysis

Implement mm_get_fragmentation_ratio() to analyze internal
fragmentation across all page families.

Closes #42
@endcode

---

@section dev_debugging Debugging

@subsection debug_symbols Build with Debug Symbols

@code{.sh}
gcc -g -O0 src/\*.c -o bin/hmm_debug
gdb bin/hmm_debug
@endcode

@subsection debug_memory Memory Debugging

**Enable verbose output:**
@code{.c}
#define MM_DEBUG 1
@endcode

**Print memory state:**
@code{.c}
mm_print_registered_page_families();
mm_print_memory_usage(MyStruct);
mm_print_vm_page_details();
@endcode

---

@section dev_advanced Advanced Topics

@subsection adv_glthread Understanding GLThread

GLThread is an intrusive linked list implementation. Each structure contains the list node:

@code{.c}
typedef struct {
int data;
glthread_t list_node; // Intrusive list node
} MyStruct;

// Convert from list node to structure
MyStruct \*s = GLTHREAD_TO_STRUCT(node, MyStruct, list_node);
@endcode

@subsection adv_page_families Page Families

Each data type gets its own "page family":

- Pages are allocated from the kernel
- Each page is divided into fixed-size blocks
- Free blocks are tracked in a priority list

@subsection adv_block_meta Block Metadata

Each allocated block has metadata:
@code{.c}
typedef struct {
bool is*free;
uint32_t block_size;
uint32_t offset;
glthread_t priority_thread_glue;
struct vm_page* \*vm_page;
} block_meta_data_t;
@endcode

---

@section dev_extending Extending the Library

@subsection ext_custom_allocator Custom Allocators

You can implement custom allocation strategies:

@code{.c}
void\* my_custom_alloc(size_t size) {
// Custom allocation logic
return mm_allocate_free_data_block(family);
}
@endcode

@subsection ext_hooks Memory Hooks

Add callbacks for allocation/deallocation events:

@code{.c}
void on_alloc(void \*ptr, size_t size) {
printf("Allocated %zu bytes at %p\\n", size, ptr);
}
@endcode

---

@section dev_resources Resources

- @ref api_reference "API Reference" - Complete function documentation
- @ref arch_overview "Architecture" - System design details
- @ref testing_guide "Testing Guide" - How to write and run tests
- [GitHub Repository](https://github.com/orcalinux/linux-memory-manager)
- [Issue Tracker](https://github.com/orcalinux/linux-memory-manager/issues)
