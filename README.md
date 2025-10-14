@mainpage Linux Memory Manager

@image html heap.png "Heap Memory Manager Architecture" width=600px

@section intro_sec Introduction

Memory management is a critical aspect of system programming, particularly in environments where resource efficiency and performance are paramount. This project presents a custom **Heap Memory Manager (HMM)** designed specifically for Linux systems, offering an alternative to the standard `malloc` and `free` functions with more efficient, flexible, and robust memory management capabilities.

@section quick_start Quick Start

@subsection for_users For Users

If you want to use this library in your application:

- @ref user_guide "User Guide" - How to integrate the library
- @ref api_reference "API Reference" - Complete API documentation
- @ref glthreads_guide "GLThreads Guide" - Understanding intrusive linked lists

@subsection for_developers For Developers

If you want to understand or contribute to the implementation:

- @ref arch_overview "Architecture Overview" - System design and internals
- @ref dev_guide "Developer Guide" - Build, test, and contribute
- @ref doxygen_guide "Doxygen Guide" - Documentation standards

---

@section why_custom Why Custom Memory Management?

In typical Linux applications, memory management is handled by the system's allocator, which uses functions like `malloc`, `calloc`, and `free`. While these standard functions are sufficient for many applications, they may not provide the level of control, efficiency, or customization required for certain high-performance or specialized applications.

This project addresses these limitations by offering:

- **Improved Performance** - Custom memory allocation algorithms can reduce fragmentation and enhance performance
- **Thread Safety** - The GLib Thread (GLThread) module provides a thread-safe linked list implementation, essential for concurrent applications
- **Memory Usage Optimization** - The HMM system is designed to make better use of available memory, reducing wastage and improving overall application efficiency
- **Fine-grained Control** - Developers have precise control over memory allocation strategies

---

@section key_features Key Features

- **Custom Heap Memory Manager** - A robust memory management system that replaces standard dynamic memory allocation
- **Thread-Safe Operations** - Generic, thread-safe linked list implementation using GLThread
- **Comprehensive Testing** - Extensive test suite ensuring reliability and robustness
- **Flexible Integration** - Available as both static (`.a`) and shared (`.so`) libraries
- **Zero External Dependencies** - Self-contained implementation for maximum portability
- **Production Ready** - Battle-tested and optimized for real-world applications

---

@section getting_started Getting Started

@subsection prerequisites Prerequisites

- GCC compiler (version 7.0 or higher)
- GNU Make
- Linux operating system (tested on Ubuntu 20.04+)
- Doxygen (optional, for documentation generation)

@subsection build_instructions Build Instructions

**Build everything:**
@code{.sh}
make all
@endcode

**Build static library only:**
@code{.sh}
make static
@endcode

**Build shared library only:**
@code{.sh}
make shared
@endcode

**Run tests:**
@code{.sh}
./bin/hmm
@endcode

**Generate documentation:**
@code{.sh}
make doc
@endcode

---

@section project_structure Project Structure

<div class="tree-container" style="background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); color: white; padding: 30px; border-radius: 12px; margin: 25px 0;">
  <div style="text-align: center; margin-bottom: 25px;">
    <h3 style="color: white; margin: 0; font-size: 20px;">📁 Linux Memory Manager</h3>
    <p style="color: rgba(255,255,255,0.9); margin: 5px 0 0 0; font-size: 14px;">Project Directory Structure</p>
  </div>
  
  <!-- Root -->
  <div class="tree-node" style="background: rgba(255,255,255,0.95); color: #2d3748; border-left: 4px solid #fbbf24; margin: 10px 0;">
    <span style="color: #fbbf24; font-weight: bold;">📦</span>
    <strong>Root Directory</strong>
  </div>
  
  <!-- src/ -->
  <div class="tree-node" style="background: rgba(255,255,255,0.95); color: #2d3748; margin-left: 20px; border-left: 4px solid #48bb78;">
    <span style="color: #48bb78; font-weight: bold;">📂</span>
    <strong>src/</strong>
    <span style="color: #718096; font-size: 13px;"> — Core implementation</span>
  </div>
  <div style="margin-left: 40px;">
    <div class="tree-node" style="background: white; color: #2d3748; border-left: 3px solid #667eea;">
      <span style="color: #667eea;">📄</span>
      <code>memory_manager.c</code>
      <span style="color: #718096; font-size: 12px;"> — Memory manager core</span>
    </div>
    <div class="tree-node" style="background: white; color: #2d3748; border-left: 3px solid #667eea;">
      <span style="color: #667eea;">📄</span>
      <code>glthread.c</code>
      <span style="color: #718096; font-size: 12px;"> — Thread-safe linked lists</span>
    </div>
    <div class="tree-node" style="background: white; color: #2d3748; border-left: 3px solid #667eea;">
      <span style="color: #667eea;">📄</span>
      <code>datatype_size_lookup.c</code>
      <span style="color: #718096; font-size: 12px;"> — Type size utilities</span>
    </div>
    <div class="tree-node" style="background: white; color: #2d3748; border-left: 3px solid #667eea;">
      <span style="color: #667eea;">📄</span>
      <code>parse_datatype.c</code>
      <span style="color: #718096; font-size: 12px;"> — Type parsing utilities</span>
    </div>
  </div>
  
  <!-- include/ -->
  <div class="tree-node" style="background: rgba(255,255,255,0.95); color: #2d3748; margin-left: 20px; border-left: 4px solid #ed8936;">
    <span style="color: #ed8936; font-weight: bold;">📂</span>
    <strong>include/</strong>
    <span style="color: #718096; font-size: 13px;"> — Public headers</span>
  </div>
  <div style="margin-left: 40px;">
    <div class="tree-node" style="background: white; color: #2d3748; border-left: 3px solid #667eea;">
      <span style="color: #667eea;">📄</span>
      <code>memory_manager_api.h</code>
      <span style="color: #718096; font-size: 12px;"> — Public API</span>
    </div>
    <div class="tree-node" style="background: white; color: #2d3748; border-left: 3px solid #667eea;">
      <span style="color: #667eea;">📄</span>
      <code>memory_manager.h</code>
      <span style="color: #718096; font-size: 12px;"> — Internal structures</span>
    </div>
    <div class="tree-node" style="background: white; color: #2d3748; border-left: 3px solid #667eea;">
      <span style="color: #667eea;">📄</span>
      <code>glthread.h</code>
      <span style="color: #718096; font-size: 12px;"> — GLThread API</span>
    </div>
    <div class="tree-node" style="background: white; color: #2d3748; border-left: 3px solid #667eea;">
      <span style="color: #667eea;">📄</span>
      <code>colors.h</code>
      <span style="color: #718096; font-size: 12px;"> — Terminal colors</span>
    </div>
  </div>
  
  <!-- docs/ -->
  <div class="tree-node" style="background: rgba(255,255,255,0.95); color: #2d3748; margin-left: 20px; border-left: 4px solid #9f7aea;">
    <span style="color: #9f7aea; font-weight: bold;">📂</span>
    <strong>docs/</strong>
    <span style="color: #718096; font-size: 13px;"> — Documentation</span>
  </div>
  <div style="margin-left: 40px;">
    <div class="tree-node" style="background: white; color: #2d3748; border-left: 3px solid #9f7aea;">
      <span style="color: #9f7aea;">📂</span>
      <code>pages/</code>
      <span style="color: #718096; font-size: 12px;"> — Additional pages</span>
    </div>
    <div class="tree-node" style="background: white; color: #2d3748; border-left: 3px solid #9f7aea;">
      <span style="color: #9f7aea;">📂</span>
      <code>assets/</code>
      <span style="color: #718096; font-size: 12px;"> — Images and resources</span>
    </div>
  </div>
  
  <!-- bin/ -->
  <div class="tree-node" style="background: rgba(255,255,255,0.95); color: #2d3748; margin-left: 20px; border-left: 4px solid #38b2ac;">
    <span style="color: #38b2ac; font-weight: bold;">📂</span>
    <strong>bin/</strong>
    <span style="color: #718096; font-size: 13px;"> — Compiled executables</span>
  </div>
  
  <!-- lib/ -->
  <div class="tree-node" style="background: rgba(255,255,255,0.95); color: #2d3748; margin-left: 20px; border-left: 4px solid #f56565;">
    <span style="color: #f56565; font-weight: bold;">📂</span>
    <strong>lib/</strong>
    <span style="color: #718096; font-size: 13px;"> — Generated libraries</span>
  </div>
  
  <!-- Makefile -->
  <div class="tree-node" style="background: rgba(255,255,255,0.95); color: #2d3748; margin-left: 20px; border-left: 4px solid #4299e1;">
    <span style="color: #4299e1;">⚙️</span>
    <strong>Makefile</strong>
    <span style="color: #718096; font-size: 13px;"> — Build system</span>
  </div>
</div>

---

@section navigation Documentation Navigation

This documentation is organized into the following sections:

**For Users:**

- @subpage user_guide
- @subpage api_reference
- @subpage integration_guide

**For Developers:**

- @subpage arch_overview
- @subpage dev_guide
- @subpage testing_guide

**Additional Resources:**

- @ref glthread_page "GLThread Data Structures"
- @ref building_guide "Building the Project"
- @ref contributing_guide "Contributing Guidelines"

---

@section license License

This project is licensed under the MIT License. See the LICENSE file for details.

@section acknowledgments Acknowledgments

This project is part of the Linux System Programming course offered by STMicroelectronics Egypt. Special thanks to the course instructors and contributors for their valuable input and guidance.

@section contact Contact & Contributing

We welcome contributions! Please see the @ref contributing_guide for details on how to contribute.

- **Issues:** Report bugs or request features on [GitHub Issues](https://github.com/orcalinux/linux-memory-manager/issues)
- **Pull Requests:** Submit improvements via pull requests
- **Documentation:** Help improve documentation
