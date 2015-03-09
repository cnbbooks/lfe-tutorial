## Macros

[forthcoming]

### The backquote macro

From the [Converting Temperature](sequential/example.md) example:

```lisp
(defun f->c
  ((`#(,name #(C ,temp)))
    ;; No conversion needed
    `#(,name #(C ,temp)))
  ((`#(,name #(F ,temp)))
    ;; Do the conversion
    `#(,name #(C ,(/ (* (- temp 32) 5) 9)))))

(defun print-temp
  ((`#(,name #(C ,temp)))
    (lfe_io:format "~-15w ~w C~n" `(,name ,temp))))
```
