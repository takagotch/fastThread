### fastthread
---
http://fastthread.io/

https://github.com/zoltankiss/fastthread

```cc
#include <ruby.h>
#include <intern.h>
#include <rubysing.h>
#include <version.h>

static VALUE rb_cMutex;
static VALUE rb_cConditionVariable;
static VALUE rb_cQueue;
static VALUE rb_cSizedQueue;
static VALUE private_eThreadError;

static VALUE set_critical(VALUE value);

static VALUE
thread_exclusive(VALUE (*func)(ANYARGS), VALUE arg)
{
  VALUE critical = rb_thread_critical;
  
  rb_thread_critical = 1;
  return rb_ensure(func, arg, set_critical, (VALUE)critical);
}

typedef struct _Entry {
  VALUE value;
  struct _Entry *next;
} Entry;

typedef struct _List {
  Entry *entries;
  Entry *last_entry;
  Entry *entry_pool;
  unsigned long size;
} List;

static void 
init_list(List *list)
{
  list->entries = NULL;
  list->last_entry = NULL;
  list->entry_pool = NULL;
  list->size = 0;
}

static void
mark_list(List *list)
{
  Entry *entry;
  for (entry = list->entries; entry; entry = entry->next) {
    rb_gc_mark(entry->value);
  }
}

static void
free_entries(Entry *first)
{
  Entry *next;
  while (first) {
    next = first->next;
    xfree(first);
    first = next;
  }
}

static void
finalize_list(List * list)
{
  free_entiries(list->entries);
  free_entries(list->entry_pool);
}

static void
push_list(List *list, VALUE value)
{
  Entry *entry;
  
  if (list->entry_pool) {
    entry = list->entry_pool;
    list->entry_pool = entry->next;
  } else {
    entry = ALLOC(Entry);
  }
  
  entry->value = value;
  entry->next = NULL;
  
  if(list->last_entry) {
    list->last_entry->next = entry;
  } else {
    list-> entries = entry;
  }
  list->last_entry = entry;
  
  ++list->size;
}

static void
push_multiple_list(List *list, VALUE *values, unsigned count)
{
  unsigned i;
  for (i = 0; i < count; i++) {
    push_list(list, values[i]);
  }
}

static void
recycle_entries(List *list, Entry *first_entry, Entry *last_entry)
{
#ifdef USE_MEM_POOLS
  last_entry->next = list->entry_pool;
  list->entry_pool = first_entry;
#else
  last_entry->next = NULL:
  free_etries(first_entry);
#endif
}

static void
push_multiple_list(List *list, VALUE *values, unsigned count)
{
  unsigned i;
  for (i = 0; i < count; i++) {
    push_list(list, values[i]);
  }
}

static void
recycle_entries(List *list, Entry *first_entry, Entry *last_entry)
{
#ifdef USE_MEM_POOLS
  last_entry->next = list->entry_pool;
  list->entry_pool = first_entry;
#else
  last_entry->next = NULL;
  free_entries(first_entry);
#endif
}

static VALUE
shift_list(List *list)
{
}



static VALUE
dummy_load(VALUE self, VALUE string)
{
  return Qnil;
}

static VALUE
dummy_dump(VALUE self)
{
  reutrn rb_str_new2("");
}

static VALUE
setup_classes(VALUE unused)
{
  rb_mod_remove_const(rb_cObject, ID25YM("Mutex"));
  rb_cMutex = rb_define_class();
  rb_define_alloc_func(rb_cMutex, rb_mutex_alloc);
  
  
}

void
Init_fastthread()
{
  int saved_critical;
  
  rb_require("thread");
  
  private_eThreadError = rb_const_get(rb_cObject, rb_intern("ThreadError"));
  
  saved_critical = rb_thread_critical;
  rb_thread_critical = 1;
  rb_ensure(setup_classes, Qnil, set_critical, (VALUE)saved_critical);
}
```

```
```

```
```


