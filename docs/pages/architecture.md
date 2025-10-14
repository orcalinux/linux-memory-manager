@page arch_overview Architecture Overview

@tableofcontents

@section arch_intro System Architecture

The Linux Memory Manager is designed with a layered architecture that provides efficient memory management while maintaining simplicity and clarity.

---

@section arch_layers Architecture Layers

<div class="arch-diagram">
  <div class="arch-layer">
    <strong>Application Layer</strong>
    <br/>User Code using XCALLOC/XFREE
  </div>
  <div class="arch-arrow">↓</div>
  <div class="arch-layer">
    <strong>Public API Layer</strong>
    <br/>memory_manager_api.h - MM_REG_STRUCT
  </div>
  <div class="arch-arrow">↓</div>
  <div class="arch-layer">
    <strong>Memory Manager Core</strong>
    <br/>Page Families, Block Allocation, Free Lists
  </div>
  <div class="arch-arrow">↓</div>
  <div class="arch-layer">
    <strong>Data Structure Layer</strong>
    <br/>GLThread Linked Lists
  </div>
  <div class="arch-arrow">↓</div>
  <div class="arch-layer">
    <strong>Kernel Layer</strong>
    <br/>mmap/munmap system calls
  </div>
</div>

---

@section arch_core_components Core Components

@subsection comp_page_family Page Family System

Each registered data type gets its own **page family**:

- Manages a set of 4KB virtual memory pages
- Each page is divided into fixed-size blocks
- Tracks free and used blocks
- Optimizes for minimal fragmentation

**Structure:**
@code{.c}
typedef struct vm*page_family* {
char struct_name[MM_MAX_STRUCT_NAME];
uint32_t struct_size;
glthread_t free_block_priority_list_head;
vm_page_for_families_t \*first_page;
} vm_page_family_t;
@endcode

@subsection comp_vm_page Virtual Memory Pages

Pages are allocated from the kernel and managed by the memory manager:

@code{.c}
typedef struct vm*page* {
struct vm*page* *next;
struct vm*page* *prev;
struct vm*page_family* \*pg_family;
uint32_t block_size;
char page_memory[0]; // Flexible array
} vm_page_t;
@endcode

**Page Layout:**

<div class="memory-block">
  <div class="memory-header">4096 bytes (SYSTEM_PAGE_SIZE)</div>
  <div class="memory-section">
    <strong>vm_page_t metadata</strong>
  </div>
  <div class="memory-data">
    <div style="display: flex; border-bottom: 1px solid #cbd5e0; padding: 5px;">
      <div style="flex: 1; border-right: 1px solid #cbd5e0; padding: 5px;">Block Meta</div>
      <div style="flex: 2; padding: 5px;">Data Block</div>
    </div>
    <div style="display: flex; border-bottom: 1px solid #cbd5e0; padding: 5px;">
      <div style="flex: 1; border-right: 1px solid #cbd5e0; padding: 5px;">Block Meta</div>
      <div style="flex: 2; padding: 5px;">Data Block</div>
    </div>
    <div style="display: flex; padding: 5px;">
      <div style="flex: 1; border-right: 1px solid #cbd5e0; padding: 5px;">...</div>
      <div style="flex: 2; padding: 5px;">...</div>
    </div>
  </div>
</div>

@subsection comp_block_meta Block Metadata

Each allocated block has associated metadata:

@code{.c}
typedef struct block*meta_data* {
bool is*free;
uint32_t block_size;
uint32_t offset;
glthread_t priority_thread_glue;
struct vm_page* \*vm_page;
} block_meta_data_t;
@endcode

**Block Allocation:**
@code
Application calls XCALLOC
↓
Find page family
↓
Get free block from priority list
↓
Mark block as used
↓
Return pointer to data section
@endcode

---

@section arch_glthread GLThread Design

GLThread is an **intrusive linked list** - the list node is embedded in the structure itself.

**Advantages:**

- Zero allocation overhead
- Cache-friendly
- Type-safe conversion
- No pointer indirection

**Example:**
@code{.c}
typedef struct {
int data;
glthread_t list_node; // Embedded node
} MyStruct;

// Add to list
MyStruct obj;
init_glthread(&obj.list_node);
glthread_add_next(&list_head, &obj.list_node);

// Traverse list
GLTHREAD_FOREACH(&list_head, node) {
MyStruct \*s = GLTHREAD_TO_STRUCT(node, MyStruct, list_node);
// Use s->data
}
@endcode

---

@section arch_memory_flow Memory Allocation Flow

@subsection flow_allocation Allocation Flow

<div class="flow-container">
  <div class="flow-step">
    <strong>Step 1:</strong> Application calls <code>Employee *e = XCALLOC(1, Employee);</code>
  </div>
  <div class="flow-arrow">↓</div>
  <div class="flow-step">
    <strong>Step 2:</strong> Lookup page family for "Employee"
  </div>
  <div class="flow-arrow">↓</div>
  <div class="flow-step">
    <strong>Step 3:</strong> Check free block list (priority ordered)
  </div>
  <div class="flow-arrow">↓</div>
  <div class="flow-step">
    <strong>Step 4:</strong> If free block available:
    <ul style="margin: 10px 0 0 20px;">
      <li>Remove from free list</li>
      <li>Mark as used</li>
      <li>Return pointer</li>
    </ul>
  </div>
  <div class="flow-arrow">↓</div>
  <div class="flow-step">
    <strong>Step 5:</strong> If no free blocks:
    <ul style="margin: 10px 0 0 20px;">
      <li>Allocate new VM page (4KB)</li>
      <li>Split into blocks</li>
      <li>Add blocks to free list</li>
      <li>Return first block</li>
    </ul>
  </div>
</div>

@subsection flow_deallocation Deallocation Flow

<div class="flow-container">
  <div class="flow-step">
    <strong>Step 1:</strong> Application calls <code>XFREE(e);</code>
  </div>
  <div class="flow-arrow">↓</div>
  <div class="flow-step">
    <strong>Step 2:</strong> Find block metadata (offset backward)
  </div>
  <div class="flow-arrow">↓</div>
  <div class="flow-step">
    <strong>Step 3:</strong> Mark block as free
  </div>
  <div class="flow-arrow">↓</div>
  <div class="flow-step">
    <strong>Step 4:</strong> Add to free block priority list
  </div>
  <div class="flow-arrow">↓</div>
  <div class="flow-step">
    <strong>Step 5:</strong> Try to merge with adjacent free blocks
  </div>
  <div class="flow-arrow">↓</div>
  <div class="flow-step">
    <strong>Step 6:</strong> If entire page is free → Return page to kernel (munmap)
  </div>
</div>

---

@section arch_optimizations Optimizations

@subsection opt_priority Priority Free Lists

Free blocks are stored in priority-ordered lists:

- Largest free blocks first
- Reduces fragmentation
- Improves allocation speed

@subsection opt_block_merging Block Merging

Adjacent free blocks are automatically merged:

- Reduces fragmentation
- Increases chance of large allocations
- Maintains memory efficiency

@subsection opt_page_recycling Page Recycling

Empty pages are returned to the kernel:

- Reduces memory footprint
- Prevents memory bloat
- Dynamic memory management

---

@section arch_thread Thread Safety Considerations

**Current Implementation:**

- GLThread operations are thread-safe
- Memory allocation is NOT thread-safe

**For Multi-threaded Applications:**

- Use external locking (mutex/spinlock)
- Consider per-thread memory pools
- Lock at page family level for finer granularity

**Example:**
@code{.c}
pthread_mutex_t mem_lock = PTHREAD_MUTEX_INITIALIZER;

void* thread_alloc(size_t size) {
pthread_mutex_lock(&mem_lock);
void *ptr = XCALLOC(1, MyStruct);
pthread_mutex_unlock(&mem_lock);
return ptr;
}
@endcode

---

@section arch_performance Performance Characteristics

<table class="perf-table">
  <thead>
    <tr>
      <th>Operation</th>
      <th>Time Complexity</th>
      <th>Notes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>XCALLOC</code></td>
      <td><span class="complexity-low">O(1) amortized</span></td>
      <td>O(n) when new page needed</td>
    </tr>
    <tr>
      <td><code>XFREE</code></td>
      <td><span class="complexity-low">O(1)</span></td>
      <td>Constant time block free</td>
    </tr>
    <tr>
      <td>Priority List Insert</td>
      <td><span class="complexity-medium">O(log n)</span></td>
      <td>Binary insertion</td>
    </tr>
    <tr>
      <td>Block Merge</td>
      <td><span class="complexity-low">O(1)</span></td>
      <td>Adjacent blocks only</td>
    </tr>
    <tr>
      <td>Page Allocation</td>
      <td><span class="complexity-low">O(1)</span></td>
      <td>mmap system call</td>
    </tr>
  </tbody>
</table>

---

@section arch_memory Memory Layout

**Typical Memory Usage:**
@code
For structure size = 64 bytes, page size = 4KB:

Page Overhead: ~128 bytes (vm_page_t)
Block Overhead: ~32 bytes per block
Usable Space: 4096 - 128 = 3968 bytes
Blocks per Page: 3968 / (64 + 32) ≈ 41 blocks

Efficiency: (41 \* 64) / 4096 ≈ 64% usable
@endcode

---

@section arch_future Future Enhancements

Potential improvements for future versions:

1. **Thread-Safe Allocator** - Lock-free or per-thread pools
2. **Memory Compaction** - Defragmentation support
3. **Allocation Statistics** - Detailed profiling
4. **Custom Page Sizes** - Configurable page sizes
5. **Memory Limits** - Quota enforcement

---

@section arch_related Related Documentation

- @ref api_reference "API Reference" - Function documentation
- @ref dev_guide "Developer Guide" - Implementation details
- @ref user_guide "User Guide" - Usage examples
