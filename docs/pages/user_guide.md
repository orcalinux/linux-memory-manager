@page user_guide User Guide

@tableofcontents

@section user_intro Introduction for Users

This guide is for developers who want to **use** the Linux Memory Manager library in their applications. If you want to understand the internal implementation or contribute to the project, see the @ref dev_guide instead.

---

@section user_installation Installation

@subsection install_from_source Installing from Source

1. **Clone the repository:**
   @code{.sh}
   git clone https://github.com/orcalinux/linux-memory-manager.git
   cd linux-memory-manager
   @endcode

2. **Build the libraries:**
   @code{.sh}
   make all
   @endcode

3. **Install (optional):**
   @code{.sh}
   sudo cp lib/libhmm.so /usr/local/lib/
   sudo cp lib/libhmm.a /usr/local/lib/
   sudo cp include/\*.h /usr/local/include/hmm/
   sudo ldconfig
   @endcode

@subsection verify_install Verify Installation

Test that the library works:
@code{.sh}
./bin/hmm
@endcode

---

@section user_integration Integration Guide

@subsection link_static Static Linking

Include the header and link with the static library:

@code{.c}
#include <hmm/memory_manager_api.h>

int main() {
mm_init();
// Your code here
return 0;
}
@endcode

Compile with:
@code{.sh}
gcc -o myapp myapp.c -I/usr/local/include -L/usr/local/lib -lhmm
@endcode

@subsection link_shared Dynamic Linking

Same code, but link dynamically:

@code{.sh}
gcc -o myapp myapp.c -I/usr/local/include -L/usr/local/lib -lhmm
export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
./myapp
@endcode

---

@section user_basic_usage Basic Usage

@subsection init_memory Initialize Memory Manager

Before using any memory allocation functions, initialize the memory manager:

@code{.c}
#include <hmm/memory_manager_api.h>

int main() {
// Initialize the memory manager
mm_init();

    // Your application code

    return 0;

}
@endcode

@subsection register_types Register Data Types

Register your custom data types with the memory manager:

@code{.c}
typedef struct {
int id;
char name[64];
float salary;
} Employee;

// Register the Employee structure
MM_REG_STRUCT(Employee);
@endcode

@subsection allocate_memory Allocate Memory

Allocate memory for your registered types:

@code{.c}
// Allocate a single Employee
Employee \*emp = XCALLOC(1, Employee);

// Allocate array of 10 Employees
Employee \*team = XCALLOC(10, Employee);

// Use the allocated memory
emp->id = 1001;
strcpy(emp->name, "John Doe");
emp->salary = 75000.0f;
@endcode

@subsection free_memory Free Memory

Free allocated memory when done:

@code{.c}
// Free single object
XFREE(emp);

// Free array
XFREE(team);
@endcode

---

@section user_advanced Advanced Usage

@subsection memory_stats Memory Statistics

Monitor memory usage in your application:

@code{.c}
// Print memory usage for a specific structure family
mm_print_memory_usage(Employee);

// Print usage for all registered structures
mm_print_registered_page_families();
@endcode

@subsection custom_allocation Custom Allocation Strategies

For specific use cases, you can control allocation:

@code{.c}
// Register a structure family with custom page size
mm_instantiate_new_page_family(
"Employee", // Structure name
sizeof(Employee) // Structure size
);

// Allocate from specific family
Employee \*emp = mm_allocate_free_data_block(
lookup_page_family_by_name("Employee")
);
@endcode

---

@section user_examples Complete Examples

@subsection example_simple Simple Application

@code{.c}
#include <stdio.h>
#include <string.h>
#include <hmm/memory_manager_api.h>

typedef struct {
int id;
char name[32];
} Student;

MM_REG_STRUCT(Student);

int main() {
// Initialize memory manager
mm_init();

    // Allocate students
    Student *student1 = XCALLOC(1, Student);
    student1->id = 101;
    strcpy(student1->name, "Alice");

    Student *student2 = XCALLOC(1, Student);
    student2->id = 102;
    strcpy(student2->name, "Bob");

    // Print memory statistics
    printf("Memory Statistics:\\n");
    mm_print_memory_usage(Student);

    // Clean up
    XFREE(student1);
    XFREE(student2);

    return 0;

}
@endcode

@subsection example_advanced Advanced Application

@code{.c}
#include <stdio.h>
#include <hmm/memory_manager_api.h>

typedef struct {
int id;
char data[100];
} Record;

MM_REG_STRUCT(Record);

int main() {
mm_init();

    // Allocate large array
    const int NUM_RECORDS = 1000;
    Record *records = XCALLOC(NUM_RECORDS, Record);

    // Initialize records
    for (int i = 0; i < NUM_RECORDS; i++) {
        records[i].id = i;
        snprintf(records[i].data, 100, "Record #%d", i);
    }

    // Check memory fragmentation
    printf("Total Memory Usage:\\n");
    mm_print_registered_page_families();

    // Free memory
    XFREE(records);

    return 0;

}
@endcode

---

@section user_best_practices Best Practices

@subsection bp_init Always Initialize

Always call mm_init() before any memory operations:
@code{.c}
int main() {
mm_init(); // First thing in main
// Rest of your code
}
@endcode

@subsection bp_register Register Before Use

Register all custom types before allocating them:
@code{.c}
MM_REG_STRUCT(MyStruct); // Register first

MyStruct \*ptr = XCALLOC(1, MyStruct); // Then allocate
@endcode

@subsection bp_matching Match Allocations and Frees

Always free what you allocate:
@code{.c}
void\* ptr = XCALLOC(1, MyType);
// Use ptr
XFREE(ptr); // Always free
@endcode

@subsection bp_null_check Check for NULL

Always check allocation results:
@code{.c}
MyType \*ptr = XCALLOC(1, MyType);
if (ptr == NULL) {
fprintf(stderr, "Allocation failed!\\n");
return -1;
}
@endcode

---

@section user_troubleshooting Troubleshooting

@subsection ts_compile Compilation Issues

**Problem:** Linker errors about undefined references

**Solution:** Make sure you're linking with the library:
@code{.sh}
gcc myapp.c -lhmm -L./lib
@endcode

**Problem:** Cannot find header files

**Solution:** Specify include path:
@code{.sh}
gcc myapp.c -I./include -lhmm
@endcode

@subsection ts_runtime Runtime Issues

**Problem:** Segmentation fault

**Solution:**

- Did you call mm_init()?
- Did you register your structure with MM_REG_STRUCT()?
- Are you freeing memory twice?

**Problem:** Memory leaks

**Solution:** Use mm_print_memory_usage() to track allocations and ensure all XCALLOC() calls have matching XFREE() calls.

---

@section user_next_steps Next Steps

- See the complete @ref api_reference for all available functions
- Check the @ref integration_guide for build system integration
- Read about @ref glthread_page for thread-safe data structures
- For internal details, see the @ref arch_overview
