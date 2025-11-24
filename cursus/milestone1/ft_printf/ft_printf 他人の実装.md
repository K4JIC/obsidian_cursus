
tayaさん
   ```c
   typedef (*function(char*, va_list*)) t_ftable;
   
   t_ftable table[128]; //ascii
   teble['c'] = *handle_c;
   teble['d'] = *handle_d;
   ```
   関数ポインタを使った実装。
   - 直感的にわかりやすく可読性が高い。
   - 配列を長めに確保するのでメモリがもったいない。
