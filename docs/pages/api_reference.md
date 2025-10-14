@page api_reference API Reference

@tableofcontents

@section api_intro API Overview

The Linux Memory Manager provides a simple, efficient API for custom memory management. All public functions are defined in `memory_manager_api.h`.

---

@section api_init Initialization

@subsection api_mm_init mm_init()

Initialize the memory manager. Must be called before any other operations.

**Declaration:**
@code{.c}
void mm_init(void);
@endcode

**Usage:**
@code{.c}
int main() {
mm_init(); // First call
// Rest of code
}
@endcode

---

@section api_allocation Memory Allocation

@subsection api_xcalloc XCALLOC()

Allocate memory for a specific type.

**Declaration:**
@code{.c}
#define XCALLOC(units, struct_name)
@endcode

**Parameters:**

- `units` - Number of elements to allocate
- `struct_name` - Type name (must be registered)

**Returns:** Pointer to allocated memory, or NULL on failure

**Example:**
@code{.c}
typedef struct {
int id;
char name[32];
} Employee;

MM_REG_STRUCT(Employee);

Employee *emp = XCALLOC(1, Employee); // Single object
Employee *team = XCALLOC(10, Employee); // Array of 10
@endcode

@subsection api_xfree XFREE()

Free previously allocated memory.

**Declaration:**
@code{.c}
#define XFREE(ptr)
@endcode

**Parameters:**

- `ptr` - Pointer to memory to free

**Example:**
@code{.c}
Employee \*emp = XCALLOC(1, Employee);
// Use emp
XFREE(emp);
@endcode

---

@section api_registration Type Registration

@subsection api_mm_reg_struct MM_REG_STRUCT()

Register a structure type with the memory manager.

**Declaration:**
@code{.c}
#define MM_REG_STRUCT(struct_name)
@endcode

**Parameters:**

- `struct_name` - Name of the structure to register

**Example:**
@code{.c}
typedef struct {
int x, y;
} Point;

MM_REG_STRUCT(Point); // Must be called before XCALLOC
@endcode

@subsection api_mm_instantiate mm_instantiate_new_page_family()

Manually create a page family for a structure.

**Declaration:**
@code{.c}
void mm_instantiate_new_page_family(
const char \*struct_name,
uint32_t struct_size
);
@endcode

**Parameters:**

- `struct_name` - Name of the structure
- `struct_size` - Size in bytes

**Example:**
@code{.c}
mm_instantiate_new_page_family("Point", sizeof(Point));
@endcode

---

@section api_statistics Memory Statistics

@subsection api_print_usage mm_print_memory_usage()

Print memory usage statistics for a specific type.

**Declaration:**
@code{.c}
#define mm_print_memory_usage(struct_name)
@endcode

**Parameters:**

- `struct_name` - Structure type to report

**Example:**
@code{.c}
mm_print_memory_usage(Employee);
@endcode

**Output:**
@code
Page Family: Employee | Size: 72 bytes
===================================
Total VM Pages: 1
Total Blocks: 56
Used Blocks: 3
Free Blocks: 53
@endcode

@subsection api_print_families mm_print_registered_page_families()

Print statistics for all registered types.

**Declaration:**
@code{.c}
void mm_print_registered_page_families(void);
@endcode

**Example:**
@code{.c}
mm_print_registered_page_families();
@endcode

@subsection api_print_page mm_print_vm_page_details()

Print detailed information about a specific memory page.

**Declaration:**
@code{.c}
void mm_print_vm_page_details(vm_page_t \*vm_page);
@endcode

---

@section api_glthread GLThread API

GLThread provides a generic, thread-safe intrusive linked list.

@subsection api_glthread_init init_glthread()

Initialize a GLThread node.

**Declaration:**
@code{.c}
void init_glthread(glthread_t \*glthread);
@endcode

@subsection api_glthread_add glthread_add_next()

Add a node after another node.

**Declaration:**
@code{.c}
void glthread_add_next(
glthread_t *base_glthread,
glthread_t *new_glthread
);
@endcode

@subsection api_glthread_remove remove_glthread()

Remove a node from the list.

**Declaration:**
@code{.c}
void remove_glthread(glthread_t \*glthread);
@endcode

@subsection api_glthread_macro GLTHREAD_TO_STRUCT()

Convert GLThread node pointer to structure pointer.

**Declaration:**
@code{.c}
#define GLTHREAD_TO_STRUCT(fn_name, structure_name, field_name, glthreadptr)
@endcode

**Example:**
@code{.c}
typedef struct {
int data;
glthread_t list_node;
} MyStruct;

GLTHREAD_TO_STRUCT(get_my_struct, MyStruct, list_node, glthread_ptr)

MyStruct \*s = get_my_struct(node_ptr);
@endcode

---

@section api_advanced Advanced Functions

@subsection api_lookup lookup_page_family_by_name()

Find a registered page family by name.

**Declaration:**
@code{.c}
vm_page_family_t* lookup_page_family_by_name(const char *struct_name);
@endcode

@subsection api_alloc_block mm_allocate_free_data_block()

Allocate a single block from a page family.

**Declaration:**
@code{.c}
void* mm_allocate_free_data_block(vm_page_family_t *vm_page_family);
@endcode

@subsection api_free_block mm_free_blocks()

Free a previously allocated block.

**Declaration:**
@code{.c}
void mm_free_blocks(void \*app_data);
@endcode

---

@section api_constants Constants and Macros

| Constant                                  | Value | Description                  |
| ----------------------------------------- | ----- | ---------------------------- |
| `SYSTEM_PAGE_SIZE`                        | 4096  | Size of VM page in bytes     |
| `MM_MAX_STRUCT_NAME`                      | 32    | Max length of structure name |
| `OFFSET_OF(type, field)`                  | -     | Get byte offset of field     |
| `MM_GET_PAGE_FROM_META_BLOCK(block_meta)` | -     | Get page from block metadata |

---

@section api_error Error Handling

Most functions return NULL on failure. Always check return values:

@code{.c}
Employee \*emp = XCALLOC(1, Employee);
if (emp == NULL) {
fprintf(stderr, "Allocation failed\\n");
return -1;
}
@endcode

---

@section api_thread Thread Safety

- GLThread operations are thread-safe
- Memory allocation functions are NOT thread-safe by default
- For multi-threaded applications, use external synchronization

@code{.c}
pthread_mutex_t mem_lock = PTHREAD_MUTEX_INITIALIZER;

pthread_mutex_lock(&mem_lock);
Employee \*emp = XCALLOC(1, Employee);
pthread_mutex_unlock(&mem_lock);
@endcode

---

@section api_examples Usage Examples

See complete examples in:

- @ref user_guide "User Guide"
- @ref example_simple "Simple Example"
- @ref example_advanced "Advanced Example"
