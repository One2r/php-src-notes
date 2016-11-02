#关于zval

PHP变量的值存储到zval结构体中,[zval结构体定义](https://github.com/php/php-src/blob/master/Zend/zend_types.h#L101-L143)在Zend/zend_types.h文件。 

    typedef struct _zval_struct zval;
    
    typedef union _zend_value {
        zend_long         lval;				/* long value */
    	double            dval;				/* double value */
    	zend_refcounted  *counted;
    	zend_string      *str;
    	zend_array       *arr;
    	zend_object      *obj;
    	zend_resource    *res;
    	zend_reference   *ref;
    	zend_ast_ref     *ast;
    	zval             *zv;
    	void             *ptr;
    	zend_class_entry *ce;
    	zend_function    *func;
    	struct {
    		uint32_t w1;
    		uint32_t w2;
    	} ww;
    } zend_value;
    
    struct _zval_struct {
        zend_value        value;			/* value */
    	union {
    		struct {
    			ZEND_ENDIAN_LOHI_4(
    				zend_uchar    type,			/* active type */
    				zend_uchar    type_flags,
    				zend_uchar    const_flags,
    				zend_uchar    reserved)	    /* call info for EX(This) */
    		} v;
    		uint32_t type_info;
    	} u1;
    	union {
    		uint32_t     next;                 /* hash collision chain */
    		uint32_t     cache_slot;           /* literal cache slot */
    		uint32_t     lineno;               /* line number (for ast nodes) */
    		uint32_t     num_args;             /* arguments number for EX(This) */
    		uint32_t     fe_pos;               /* foreach position */
    		uint32_t     fe_iter_idx;          /* foreach iterator index */
    		uint32_t     access_flags;         /* class constant access flags */
    		uint32_t     property_guard;       /* single property guard */
    	} u2;
    };
    
用[如下宏](https://github.com/php/php-src/blob/master/Zend/zend_types.h#L340-L341)可以判断zval变量类型：

	/* we should never set just Z_TYPE, we should set Z_TYPE_INFO */
	#define Z_TYPE(zval)				zval_get_type(&(zval))
	#define Z_TYPE_P(zval_p) 			Z_TYPE(*(zval_p))

[zval_get _type](https://github.com/php/php-src/blob/master/Zend/zend_types.h#L330-L332)方法返回的是_zval_struct结构体中u1.v.type存储的[变量类型值](https://github.com/php/php-src/blob/master/Zend/zend_types.h#L302-L313)

	static zend_always_inline zend_uchar zval_get_type(const zval* pz) {
		return pz->u1.v.type;
	}

判断zval变量类型的例子：

	if (Z_TYPE(retval) == IS_TRUE) {
			ret = SUCCESS;
	}