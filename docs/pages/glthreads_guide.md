@page glthreads_guide GLThreads Data Structures Guide

@tableofcontents

@section glthread_intro Introduction

**GLThreads** (Generic Linked Threads) is a powerful programming technique used to manage data structures with enhanced flexibility and memory efficiency. GLThreads allow data structures to embed nodes directly within them, making it possible for a single data structure to participate in multiple linked structures simultaneously.

<div class="info-box">
<strong>💡 Key Insight:</strong> GLThreads is particularly useful in low-level systems programming, where performance and memory efficiency are critical.
</div>

---

@section glthread_why Why It's Called GLThreads

The term **GLThreads** or **Generic Linked Threads** originates from the concept of "threads" in the context of linked lists and data structures.

<div class="feature-grid">
  <div class="feature-card">
    <div class="feature-title">🔗 "Thread" Metaphor</div>
    <div class="feature-desc">
      Just as a thread in sewing connects fabric pieces, GLThreads create a "thread" of connections linking elements together in a data structure.
    </div>
  </div>
  
  <div class="feature-card">
    <div class="feature-title">🔀 "Linked" Component</div>
    <div class="feature-desc">
      Embedded nodes connect elements within data structures, allowing participation in multiple linked lists or other structures simultaneously.
    </div>
  </div>
  
  <div class="feature-card">
    <div class="feature-title">⚙️ "Generic" Nature</div>
    <div class="feature-desc">
      Not limited to any specific type of data structure—applicable to linked lists, queues, trees, and more.
    </div>
  </div>
</div>

---

@section glthread_concepts Key Concepts

@subsection concept_embedded Embedded Nodes

Instead of using standalone nodes with pointers to the next and previous elements, GLThreads embed these nodes directly within the data structure. This allows the structure to be part of multiple lists without modification.

<div class="memory-block">
  <div class="memory-header">Traditional Node vs. GLThread Node</div>
  <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 20px; padding: 15px;">
    <div>
      <strong>❌ Traditional Approach:</strong>
      <pre style="background: #fff; padding: 10px; border-radius: 4px; margin-top: 10px;">
struct Node {
  Data* data;
  Node* next;
  Node* prev;
};</pre>
      <em style="color: #e53e3e;">Requires separate allocation</em>
    </div>
    <div>
      <strong>✅ GLThread Approach:</strong>
      <pre style="background: #fff; padding: 10px; border-radius: 4px; margin-top: 10px;">
struct Data {
  int value;
  glthread_t node;
};</pre>
      <em style="color: #38a169;">Node embedded in data</em>
    </div>
  </div>
</div>

@subsection concept_offset Offset-Based Navigation

Pointers to next and previous nodes are calculated using offsets within the containing structure. This allows for flexible management of data structures.

@code{.c}
#define OFFSET_OF(type, member) ((size_t)&(((type _)0)->member))
#define GLTHREAD_TO_STRUCT(ptr, type, member) \
 (type _)((char \*)(ptr) - OFFSET_OF(type, member))
@endcode

@subsection concept_multi Multiple List Participation

A single data structure can participate in multiple lists, queues, or other linked structures simultaneously, enabling more complex relationships between data elements.

<div class="flow-container">
  <div class="flow-step">
    <strong>Employee Structure</strong>
    <code>{ name, id, age_list_node, dept_list_node }</code>
  </div>
  <div style="display: flex; justify-content: space-around; margin: 20px 0;">
    <div class="flow-arrow" style="transform: rotate(-45deg);">↓</div>
    <div class="flow-arrow" style="transform: rotate(45deg);">↓</div>
  </div>
  <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 20px;">
    <div class="flow-step">Age-sorted List</div>
    <div class="flow-step">Department List</div>
  </div>
</div>

---

@section glthread_advantages Advantages

<div class="feature-grid">
  <div class="feature-card">
    <div class="feature-title">💾 Memory Efficiency</div>
    <div class="feature-desc">
      By embedding nodes within the data structure, GLThreads reduce memory overhead by avoiding separate allocations for each node.
    </div>
  </div>
  
  <div class="feature-card">
    <div class="feature-title">🔄 Flexibility</div>
    <div class="feature-desc">
      The same data structure can be linked into multiple lists or queues without needing to modify its definition.
    </div>
  </div>
  
  <div class="feature-card">
    <div class="feature-title">⚡ Cache Performance</div>
    <div class="feature-desc">
      Since nodes are embedded within a structure that is often accessed together, cache locality is improved.
    </div>
  </div>
</div>

---

@section glthread_usecases Use Cases

<table class="perf-table">
  <thead>
    <tr>
      <th>Domain</th>
      <th>Application</th>
      <th>Benefit</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Operating Systems</strong></td>
      <td>Managing processes or tasks in different queues (run queue, wait queue)</td>
      <td>Single process can be in multiple queues simultaneously</td>
    </tr>
    <tr>
      <td><strong>Embedded Systems</strong></td>
      <td>Efficiently managing resources with minimal overhead</td>
      <td>Reduced memory footprint and allocation overhead</td>
    </tr>
    <tr>
      <td><strong>Complex Data Management</strong></td>
      <td>Data structures participating in multiple lists</td>
      <td>Flexible relationships without data duplication</td>
    </tr>
  </tbody>
</table>

---

@section glthread_impl Implementation Examples

@subsection impl_linkedlist Linked List with GLThreads

@code{.c}
#include <stddef.h>
#include <stdio.h>

typedef struct glthread_node {
struct glthread_node *prev;
struct glthread_node *next;
} glthread_node_t;

typedef struct {
char name[32];
int id;
glthread_node_t glnode;
} employee_t;

#define OFFSET_OF(type, member) ((size_t)&(((type _)0)->member))
#define GLTHREAD_TO_STRUCT(ptr, type, member) \
 (type _)((char \*)(ptr) - OFFSET_OF(type, member))

void glthread_add_next(glthread_node_t *base_node, glthread_node_t *new_node) {
new_node->prev = base_node;
new_node->next = base_node->next;
if (base_node->next) {
base_node->next->prev = new_node;
}
base_node->next = new_node;
}

void init_glthread(glthread_node_t \*glnode) {
glnode->prev = NULL;
glnode->next = NULL;
}

int main() {
employee_t e1 = {.name = "Alice", .id = 1};
employee_t e2 = {.name = "Bob", .id = 2};

    init_glthread(&e1.glnode);
    init_glthread(&e2.glnode);

    glthread_add_next(&e1.glnode, &e2.glnode);

    // Traverse list
    glthread_node_t *current = &e1.glnode;
    while (current) {
        employee_t *emp = GLTHREAD_TO_STRUCT(current, employee_t, glnode);
        printf("Employee: %s (ID: %d)\n", emp->name, emp->id);
        current = current->next;
    }

    return 0;

}
@endcode

<div class="success-box">
<strong>✅ Output:</strong>
<pre>
Employee: Alice (ID: 1)
Employee: Bob (ID: 2)
</pre>
</div>

@subsection impl_queue Queue with GLThreads

@code{.c}
typedef struct glthread_queue {
glthread_node_t *head;
glthread_node_t *tail;
} glthread_queue_t;

void queue_init(glthread_queue_t \*queue) {
queue->head = NULL;
queue->tail = NULL;
}

void queue_enqueue(glthread_queue_t *queue, glthread_node_t *node) {
node->next = NULL;
node->prev = queue->tail;

    if (queue->tail) {
        queue->tail->next = node;
    } else {
        queue->head = node;
    }
    queue->tail = node;

}

glthread_node_t* queue_dequeue(glthread_queue_t *queue) {
if (!queue->head) return NULL;

    glthread_node_t *node = queue->head;
    queue->head = node->next;

    if (queue->head) {
        queue->head->prev = NULL;
    } else {
        queue->tail = NULL;
    }

    node->next = node->prev = NULL;
    return node;

}
@endcode

@subsection impl_tree Binary Tree with GLThreads

<div class="info-box">
<strong>🌲 Tree Structure:</strong> For binary trees, use two GLThread nodes—one for left child, one for right child.
</div>

@code{.c}
typedef struct tree_node {
int value;
glthread_node_t left_child;
glthread_node_t right_child;
} tree_node_t;

void tree_init(tree_node_t \*node, int value) {
node->value = value;
init_glthread(&node->left_child);
init_glthread(&node->right_child);
}

void tree_add_left(tree_node_t *parent, tree_node_t *child) {
parent->left_child.next = &child->left_child;
child->left_child.prev = &parent->left_child;
}

void tree_add_right(tree_node_t *parent, tree_node_t *child) {
parent->right_child.next = &child->right_child;
child->right_child.prev = &parent->right_child;
}
@endcode

---

@section glthread_advanced Advanced Examples

@subsection adv_deque Double-Ended Queue (Deque)

<div class="flow-container">
  <div class="flow-step">
    <strong>Deque Operations:</strong>
    <ul style="margin: 10px 0 0 20px;">
      <li><code>push_front()</code> - Add to head</li>
      <li><code>push_back()</code> - Add to tail</li>
      <li><code>pop_front()</code> - Remove from head</li>
      <li><code>pop_back()</code> - Remove from tail</li>
    </ul>
  </div>
</div>

@code{.c}
typedef struct deque {
glthread_node_t *head;
glthread_node_t *tail;
} deque_t;

void deque_push_front(deque_t *dq, glthread_node_t *node) {
node->prev = NULL;
node->next = dq->head;

    if (dq->head) {
        dq->head->prev = node;
    } else {
        dq->tail = node;
    }
    dq->head = node;

}

void deque_push_back(deque_t *dq, glthread_node_t *node) {
node->next = NULL;
node->prev = dq->tail;

    if (dq->tail) {
        dq->tail->next = node;
    } else {
        dq->head = node;
    }
    dq->tail = node;

}

glthread_node_t* deque_pop_front(deque_t *dq) {
if (!dq->head) return NULL;

    glthread_node_t *node = dq->head;
    dq->head = node->next;

    if (dq->head) {
        dq->head->prev = NULL;
    } else {
        dq->tail = NULL;
    }

    node->next = node->prev = NULL;
    return node;

}

glthread_node_t* deque_pop_back(deque_t *dq) {
if (!dq->tail) return NULL;

    glthread_node_t *node = dq->tail;
    dq->tail = node->prev;

    if (dq->tail) {
        dq->tail->next = NULL;
    } else {
        dq->head = NULL;
    }

    node->next = node->prev = NULL;
    return node;

}
@endcode

@subsection adv_circular Circular Linked List

<div class="tree-container">
  <div style="text-align: center; margin-bottom: 20px;">
    <strong style="color: #667eea; font-size: 16px;">↻ Circular Linked List Structure</strong>
  </div>
  <div class="tree-node" style="background: linear-gradient(90deg, #667eea 0%, #764ba2 100%); color: white;">
    <span class="tree-branch">●</span>
    <span class="tree-content">Node 1 (Head)</span>
    <span style="float: right;">→ Next | ← Prev</span>
  </div>
  <div style="text-align: center; color: #667eea; font-size: 20px;">↓ ↑</div>
  <div class="tree-node" style="background: linear-gradient(90deg, #48bb78 0%, #38a169 100%); color: white;">
    <span class="tree-branch">●</span>
    <span class="tree-content">Node 2</span>
    <span style="float: right;">→ Next | ← Prev</span>
  </div>
  <div style="text-align: center; color: #48bb78; font-size: 20px;">↓ ↑</div>
  <div class="tree-node" style="background: linear-gradient(90deg, #ed8936 0%, #dd6b20 100%); color: white;">
    <span class="tree-branch">●</span>
    <span class="tree-content">Node 3</span>
    <span style="float: right;">→ Next | ← Prev</span>
  </div>
  <div style="text-align: center; color: #ed8936; font-size: 20px;">↓ ↑</div>
  <div class="tree-node" style="background: linear-gradient(90deg, #f56565 0%, #e53e3e 100%); color: white;">
    <span class="tree-branch">●</span>
    <span class="tree-content">Node 4 (Tail)</span>
    <span style="float: right;">→ Back to Head</span>
  </div>
  <div style="text-align: center; margin-top: 15px; padding: 10px; background: rgba(102, 126, 234, 0.1); border-radius: 6px;">
    <em style="color: #667eea;">Each node points to the next, and the last node points back to the first</em>
  </div>
</div>

@code{.c}
void circular_list_add(glthread_node_t *head, glthread_node_t *new_node) {
if (!head->next) {
// First node
head->next = new_node;
head->prev = new_node;
new_node->next = head;
new_node->prev = head;
} else {
// Insert before head
new_node->next = head;
new_node->prev = head->prev;
head->prev->next = new_node;
head->prev = new_node;
}
}

void circular_list_traverse(glthread_node_t \*head) {
if (!head->next) return;

    glthread_node_t *current = head->next;
    do {
        // Process current node
        current = current->next;
    } while (current != head->next);

}
@endcode

---

@section glthread_comparison Comparison: Traditional vs GLThreads

<table class="perf-table">
  <thead>
    <tr>
      <th>Aspect</th>
      <th>Traditional Nodes</th>
      <th>GLThreads</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Memory Allocation</strong></td>
      <td>Separate allocations for each node</td>
      <td>Single allocation for data + embedded node</td>
    </tr>
    <tr>
      <td><strong>Cache Locality</strong></td>
      <td>Poor - scattered memory</td>
      <td>Good - data and node together</td>
    </tr>
    <tr>
      <td><strong>Multiple Lists</strong></td>
      <td>Requires wrapper structures or duplication</td>
      <td>Embed multiple nodes in one structure</td>
    </tr>
    <tr>
      <td><strong>Type Safety</strong></td>
      <td>Requires explicit casting</td>
      <td>Type-safe macros (GLTHREAD_TO_STRUCT)</td>
    </tr>
    <tr>
      <td><strong>Complexity</strong></td>
      <td>Simple, straightforward</td>
      <td>Requires understanding of offset calculation</td>
    </tr>
  </tbody>
</table>

---

@section glthread_bestpractices Best Practices

<div class="success-box">
<strong>✅ DO:</strong>
<ul style="margin: 10px 0 0 20px;">
  <li>Use GLThreads for structures that need to be in multiple lists</li>
  <li>Initialize glthread nodes before use</li>
  <li>Use type-safe macros (GLTHREAD_TO_STRUCT)</li>
  <li>Document which list each embedded node is for</li>
</ul>
</div>

<div class="warning-box">
<strong>⚠️ DON'T:</strong>
<ul style="margin: 10px 0 0 20px;">
  <li>Don't forget to remove nodes before freeing memory</li>
  <li>Don't use the same glthread node in multiple lists simultaneously</li>
  <li>Don't manually manipulate pointers without proper checks</li>
  <li>Don't assume thread safety without external locking</li>
</ul>
</div>

---

@section glthread_related Related Documentation

- @ref arch_overview "Architecture Overview" - System design using GLThreads
- @ref dev_guide "Developer Guide" - Implementation details
- @ref api_reference "API Reference" - GLThread API functions
