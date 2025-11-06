
#update_save

```c
t_memory *update_save(t_memory *old_save)
{
    t_memory *new_save;
    size_t new_len;
    unsigned char *nlptr;

    nlptr = ft_memchr(old_save->content, '\n', old_save->len);
    if (!nlptr)
    {
        new_save = init_memory(NULL, 0);
        return (new_save);
    }
    new_len = old_save->len - (nlptr - old_save->content + 1);
    new_save = init_memory(nlptr + 1, new_len);
    return (new_save);
}
```

