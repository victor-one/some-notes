# DPDK无锁环形缓冲

## rte_ring的数据结构定义

```c
/**
 * An RTE ring structure.
 *
 * The producer and the consumer have a head and a tail index. The particularity
 * of these index is that they are not between 0 and size(ring). These indexes
 * are between 0 and 2^32, and we mask their value when we access the ring[]
 * field. Thanks to this assumption, we can do subtractions between 2 index
 * values in a modulo-32bit base: that's why the overflow of the indexes is not
 * a problem.
 */
struct rte_ring {
	/*
	 * Note: this field kept the RTE_MEMZONE_NAMESIZE size due to ABI
	 * compatibility requirements, it could be changed to RTE_RING_NAMESIZE
	 * next time the ABI changes
	 */
	char name[RTE_MEMZONE_NAMESIZE] __rte_cache_aligned; /**< Name of the ring. */
	int flags;               /**< Flags supplied at creation. */
	const struct rte_memzone *memzone;
			/**< Memzone, if any, containing the rte_ring */
	uint32_t size;           /**< Size of ring. */
	uint32_t mask;           /**< Mask (size-1) of ring. */
	uint32_t capacity;       /**< Usable size of ring */

	char pad0 __rte_cache_aligned; /**< empty cache line */

	/** Ring producer status. */
	struct rte_ring_headtail prod __rte_cache_aligned;
	char pad1 __rte_cache_aligned; /**< empty cache line */

	/** Ring consumer status. */
	struct rte_ring_headtail cons __rte_cache_aligned;
	char pad2 __rte_cache_aligned; /**< empty cache line */
};
```

`rte_ring_headtail`结构体如下：

```c
/* structure to hold a pair of head/tail values and other metadata */
struct rte_ring_headtail {
	volatile uint32_t head;  /**< Prod/consumer head. */
	volatile uint32_t tail;  /**< Prod/consumer tail. */
	uint32_t single;         /**< True if single prod/cons */
};
```

在这个结构体中存放着生产者和消费者的 head 值和 tail 值，并且存放着一个用于判断是否为单生产者或单消费者的 single 。

## rte_ring_init

```c
int
rte_ring_init(struct rte_ring *r, const char *name, unsigned count,
	unsigned flags)
{
	int ret;

	/* compilation-time checks */
	RTE_BUILD_BUG_ON((sizeof(struct rte_ring) &
			  RTE_CACHE_LINE_MASK) != 0);
	RTE_BUILD_BUG_ON((offsetof(struct rte_ring, cons) &
			  RTE_CACHE_LINE_MASK) != 0);
	RTE_BUILD_BUG_ON((offsetof(struct rte_ring, prod) &
			  RTE_CACHE_LINE_MASK) != 0);

	/* init the ring structure */
	memset(r, 0, sizeof(*r));
	ret = snprintf(r->name, sizeof(r->name), "%s", name);
	if (ret < 0 || ret >= (int)sizeof(r->name))
		return -ENAMETOOLONG;
	r->flags = flags;
	r->prod.single = (flags & RING_F_SP_ENQ) ? __IS_SP : __IS_MP;
	r->cons.single = (flags & RING_F_SC_DEQ) ? __IS_SC : __IS_MC;

	if (flags & RING_F_EXACT_SZ) {
		r->size = rte_align32pow2(count + 1);
		r->mask = r->size - 1;
		r->capacity = count;
	} else {
		if ((!POWEROF2(count)) || (count > RTE_RING_SZ_MASK)) {
			RTE_LOG(ERR, RING,
				"Requested size is invalid, must be power of 2, and not exceed the size limit %u\n",
				RTE_RING_SZ_MASK);
			return -EINVAL;
		}
		r->size = count;
		r->mask = count - 1;
		r->capacity = r->mask;
	}
	r->prod.head = r->cons.head = 0;
	r->prod.tail = r->cons.tail = 0;

	return 0;
}
```

此函数接受四个参数，分别是指向`rte_ring`结构体的指针`*r`、常量char类型的指针`*name`、无符号类型的`count`、无符号类型的`flags`。

`RTE_BUILD_BUG_ON`的作用是判断`rte_ring`结构体的成员在缓存行上对齐，确保缓存局部性，以提高内存访问效率。

使用`memset`将结构体初始化为全0。

使用`snprintf`函数将name字符串的值复制到`r->name`中，如果返回值小于0，或超过了`r->name`字段的大小，则返回对应的错误码。

将`flags`值赋值给`r->flags`，并根据`flags`的值进行掩码运算，从而判断是否为单生产者和是否为单消费者。

根据`flags`进行`RING_F_EXACT_SZ`掩码运算，从而设置环形缓冲区中分配存储空间时精确匹配请求的大小。如果设置了，则将`r->size`设置为对应`count + 1`的下一次 2 的幂，`r->mask`设置为`r->size - 1`，`r->capacity`设置为`count`。否则，则需要对`count`进行判断，若`count`不为2的幂或超出了`RTE_RING_SZ_MASK`，则在日志中打印报错信息并返回错误码。如果判断通过，则将`r->size`设置为对应`count`，`r->mask`设置为`count - 1`，`r->capacity`设置为`r->mask`。

初始化时将生产者头和消费者头都设为0，将生产者尾和消费者尾也都设为0。

## rte_ring_create

```c
/* create the ring */
struct rte_ring *
rte_ring_create(const char *name, unsigned count, int socket_id,
		unsigned flags)
{
	char mz_name[RTE_MEMZONE_NAMESIZE];
	struct rte_ring *r;
	struct rte_tailq_entry *te;
	const struct rte_memzone *mz;
	ssize_t ring_size;
	int mz_flags = 0;
	struct rte_ring_list* ring_list = NULL;
	const unsigned int requested_count = count;
	int ret;

	ring_list = RTE_TAILQ_CAST(rte_ring_tailq.head, rte_ring_list);

	/* for an exact size ring, round up from count to a power of two */
	if (flags & RING_F_EXACT_SZ)
		count = rte_align32pow2(count + 1);

	ring_size = rte_ring_get_memsize(count);
	if (ring_size < 0) {
		rte_errno = ring_size;
		return NULL;
	}

	ret = snprintf(mz_name, sizeof(mz_name), "%s%s",
		RTE_RING_MZ_PREFIX, name);
	if (ret < 0 || ret >= (int)sizeof(mz_name)) {
		rte_errno = ENAMETOOLONG;
		return NULL;
	}

	te = rte_zmalloc("RING_TAILQ_ENTRY", sizeof(*te), 0);
	if (te == NULL) {
		RTE_LOG(ERR, RING, "Cannot reserve memory for tailq\n");
		rte_errno = ENOMEM;
		return NULL;
	}

	rte_rwlock_write_lock(RTE_EAL_TAILQ_RWLOCK);

	/* reserve a memory zone for this ring. If we can't get rte_config or
	 * we are secondary process, the memzone_reserve function will set
	 * rte_errno for us appropriately - hence no check in this this function */
	mz = rte_memzone_reserve_aligned(mz_name, ring_size, socket_id,
					 mz_flags, __alignof__(*r));
	if (mz != NULL) {
		r = mz->addr;
		/* no need to check return value here, we already checked the
		 * arguments above */
		rte_ring_init(r, name, requested_count, flags);

		te->data = (void *) r;
		r->memzone = mz;

		TAILQ_INSERT_TAIL(ring_list, te, next);
	} else {
		r = NULL;
		RTE_LOG(ERR, RING, "Cannot reserve memory\n");
		rte_free(te);
	}
	rte_rwlock_write_unlock(RTE_EAL_TAILQ_RWLOCK);

	return r;
}
```

此函数接受四个参数，分别为具有const限定符的char类型指针`*name`、unsigned类型的整数`count`、int类型的整数`socket_id`、unsigned类型的整数`flags`。

这里的`socket_id`中的`socket`指的不是unix网络编程中的socket，而是指的numa节点。

首先利用全局的`rte_ring_tailq`找到`ring_list`。

通过传递的`flags`与`RING_F_EXACT_SZ`进行按位与，从而判断是否需要将传递进来的`count`转换为最近的二的幂。

通过`count`的值获取`ring_size`，如果小于0，将此时的`ring_size`作为错误码，并返回空值。

通过`snprintf`函数，将`"RG_" + name`赋值给`mz_name`，并将写入字符串的长度赋值为`ret`。如果`ret`的值小于0或大于`mz_name`的大小，将`ENAMETOOLONG`作为错误码赋值给`rte_errno`，并返回空值。

通过`rte_zmalloc`函数为`te`分配内存，如果为空，则在日志中报错并将`ENOMEM`错误码赋值给`rte_errno`，返回空值。

`mem_config`上锁后，使用`rte_memzone_reserve_aligned`函数为这个环分配内存区域。如果成功分配了内存区域，将`mz->addr`作为`r`的值（即将指针`*r`指向分配的这段内存区域），对`rte_ring`初始化，并将`te`挂接到全局队列上。否则，将`r`设为空值，在日志中打印错误信息并释放`te`。

释放`mem_config`锁，并返回指针`r`。

## __rte_ring_do_enqueue

```c
/**
 * @internal Enqueue several objects on the ring
 *
  * @param r
 *   A pointer to the ring structure.
 * @param obj_table
 *   A pointer to a table of void * pointers (objects).
 * @param n
 *   The number of objects to add in the ring from the obj_table.
 * @param behavior
 *   RTE_RING_QUEUE_FIXED:    Enqueue a fixed number of items from a ring
 *   RTE_RING_QUEUE_VARIABLE: Enqueue as many items as possible from ring
 * @param is_sp
 *   Indicates whether to use single producer or multi-producer head update
 * @param free_space
 *   returns the amount of space after the enqueue operation has finished
 * @return
 *   Actual number of objects enqueued.
 *   If behavior == RTE_RING_QUEUE_FIXED, this will be 0 or n only.
 */
static __rte_always_inline unsigned int
__rte_ring_do_enqueue(struct rte_ring *r, void * const *obj_table,
		 unsigned int n, enum rte_ring_queue_behavior behavior,
		 unsigned int is_sp, unsigned int *free_space)
{
	uint32_t prod_head, prod_next;
	uint32_t free_entries;

  // 偏移生产者头指针，抢占生产位置
	n = __rte_ring_move_prod_head(r, is_sp, n, behavior,
			&prod_head, &prod_next, &free_entries);
	if (n == 0)
		goto end;

  // 填入数据
	ENQUEUE_PTRS(r, &r[1], prod_head, obj_table, n, void *);

  // 更新生产者尾指针
	update_tail(&r->prod, prod_head, prod_next, is_sp, 1);
end:
	if (free_space != NULL)
		*free_space = free_entries - n;
	return n;
}
```

此函数中接受六个参数，分别为：指向ring结构体的指针`*r`、指向一张对象表的指针`*obj_table`、从对象表中往ring中添加的对象的数量`n`、行为`behavior`（有两种可以选择，分别为`RTE_RING_QUEUE_FIXED`和`RTE_RING_QUEUE_VARIABLE`，对应着入队固定数量的项目和入队尽可能多的项目）、是否为单生产者`is_sp`、指向入队操作后返回空间数量的指针`free_space`。

通过`__rte_ring_move_prod_head`函数对生产者头指针进行操作，函数如下：

```c
/**
 * @internal This function updates the producer head for enqueue
 *
 * @param r
 *   A pointer to the ring structure
 * @param is_sp
 *   Indicates whether multi-producer path is needed or not
 * @param n
 *   The number of elements we will want to enqueue, i.e. how far should the
 *   head be moved
 * @param behavior
 *   RTE_RING_QUEUE_FIXED:    Enqueue a fixed number of items from a ring
 *   RTE_RING_QUEUE_VARIABLE: Enqueue as many items as possible from ring
 * @param old_head
 *   Returns head value as it was before the move, i.e. where enqueue starts
 * @param new_head
 *   Returns the current/new head value i.e. where enqueue finishes
 * @param free_entries
 *   Returns the amount of free space in the ring BEFORE head was moved
 * @return
 *   Actual number of objects enqueued.
 *   If behavior == RTE_RING_QUEUE_FIXED, this will be 0 or n only.
 */
static __rte_always_inline unsigned int
__rte_ring_move_prod_head(struct rte_ring *r, unsigned int is_sp,
		unsigned int n, enum rte_ring_queue_behavior behavior,
		uint32_t *old_head, uint32_t *new_head,
		uint32_t *free_entries)
{
	const uint32_t capacity = r->capacity;
	unsigned int max = n;
	int success;

	do {
		/* Reset n to the initial burst count */
		n = max;

    // 拿到现在生产者的头位置，也即即将生产的位置
		*old_head = r->prod.head;

		/* add rmb barrier to avoid load/load reorder in weak
		 * memory model. It is noop on x86
		 */
		rte_smp_rmb();

		/*
		 *  The subtraction is done between two unsigned 32bits value
		 * (the result is always modulo 32 bits even if we have
		 * *old_head > cons_tail). So 'free_entries' is always between 0
		 * and capacity (which is < size).
		 */
		*free_entries = (capacity + r->cons.tail - *old_head);

		/* check that we have enough room in ring */
		if (unlikely(n > *free_entries))
			n = (behavior == RTE_RING_QUEUE_FIXED) ?
					0 : *free_entries;

		if (n == 0)
			return 0;

    // 计算生产之后的新位置
		*new_head = *old_head + n;
		if (is_sp)
			r->prod.head = *new_head, success = 1;
		else
      // 如果是多生产者调用cmpset函数实现生产位置抢占
			success = rte_atomic32_cmpset(&r->prod.head,
					*old_head, *new_head);
	} while (unlikely(success == 0));
	return n;
}
```

首先获取ring的可用大小，将`max`的初始值设置为`n`，之后进入循环。

每次循环时，首先将`n`的值设为`max`，将当前生产者头部指针`r->prod.head`存储在`old_head`指向的变量中。

添加一个内存屏障以保证正确的次序。

通过`*free_entries = (capacity + r->cons.tail - *old_head);`计算出当前可用空间的数量。

如果可用空间数量不足，如果`behavior`为`RTE_RING_QUEUE_FIXED`，则将`n`设为0，否则设为`*free_entries`。

如果`n`为0，则函数返回0。

将`*new_head`设为`*old_head + n`。

如果`is_sp`为真，则将`r->prod.head`设为`*new_head`，并将`success`置为1。否则使用`rte_atomic32_cmpset`来设置新的生产者头部指针，并将结果返回给`success`。

这个过程会一直循环，直到`success`被设为0。

之后返回`n`。



对生产者尾指针的操作使用`update_tail`函数，函数如下：

```c
static __rte_always_inline void
update_tail(struct rte_ring_headtail *ht, uint32_t old_val, uint32_t new_val,
		uint32_t single, uint32_t enqueue)
{
	if (enqueue)
		rte_smp_wmb();
	else
		rte_smp_rmb();
	/*
	 * If there are other enqueues/dequeues in progress that preceded us,
	 * we need to wait for them to complete
	 */
	if (!single)
		while (unlikely(ht->tail != old_val))
			rte_pause();

	ht->tail = new_val;
}
```

此函数中接受五个参数，分别为用于存放头尾值的结构体指针`*ht`、原来的值`old_val`、新的值`new_val`、用于判断是否为单生产者的`single`、判断是否入队情况的`enqueue`。

如果是入队情况需设置写内存屏障，否则需设置读内存屏障。

如果为多生产者/消费者情况，需等待其它生产者/消费者更新tail指针后，本生产者/消费者才可以对tail指针进行操作。

更新tail指针至最新的位置，使刚生产的数据可以全部被消费者消费。

## __rte_ring_do_dequeue

```c
/**
 * @internal Dequeue several objects from the ring
 *
 * @param r
 *   A pointer to the ring structure.
 * @param obj_table
 *   A pointer to a table of void * pointers (objects).
 * @param n
 *   The number of objects to pull from the ring.
 * @param behavior
 *   RTE_RING_QUEUE_FIXED:    Dequeue a fixed number of items from a ring
 *   RTE_RING_QUEUE_VARIABLE: Dequeue as many items as possible from ring
 * @param is_sc
 *   Indicates whether to use single consumer or multi-consumer head update
 * @param available
 *   returns the number of remaining ring entries after the dequeue has finished
 * @return
 *   - Actual number of objects dequeued.
 *     If behavior == RTE_RING_QUEUE_FIXED, this will be 0 or n only.
 */
static __rte_always_inline unsigned int
__rte_ring_do_dequeue(struct rte_ring *r, void **obj_table,
		 unsigned int n, enum rte_ring_queue_behavior behavior,
		 unsigned int is_sc, unsigned int *available)
{
	uint32_t cons_head, cons_next;
	uint32_t entries;

	n = __rte_ring_move_cons_head(r, (int)is_sc, n, behavior,
			&cons_head, &cons_next, &entries);
	if (n == 0)
		goto end;

	DEQUEUE_PTRS(r, &r[1], cons_head, obj_table, n, void *);

	update_tail(&r->cons, cons_head, cons_next, is_sc, 0);

end:
	if (available != NULL)
		*available = entries - n;
	return n;
}

```

