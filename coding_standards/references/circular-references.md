# Circular References

> A circular reference is a series of references where the last object
> references the first, resulting in a closed loop.

[Circular references](#circular-references) are a well knowng issue with PHP and its garbage collector;
this because PHP usesa reference counted memory allocation mechanism for its
internal variables.

By default the PHP garbage collector cannot handle circular references in
complex objects (like Magento collections), this results in the allocated
memory for those objects to not bee freed.

When working with collections that can potentially have hundres of thousands of
objects, it will quickly run out of memory.
