get-addr-of(v)
set-val-at-addr(a, v)
get-val-at-addr(a)

rval(v) => get-val-at-addr(get-addr-of(v))
lval(l,r) => set-val-at-addr(get-addr-of(l), get-val-at-addr(get-addr-of(r)))

x = *y;  // get-val-at-addr(get-val-at-addr(get-addr-of(y)))
*x = y;  // set-val-at-addr(get-val-at-addr(get-addr-of(x)))
**x = y; // set-val-at-addr(get-val-at-addr(get-val-at-addr(get-addr-of(x))))
x = **y; // get-val-at-addr(get-val-at-addr(get-val-at-addr(get-addr-of(y))))