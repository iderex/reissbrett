A tracked text file whose stored bytes carry CRLF.

It exists to red the text-determinism check and nothing else. The blob was
written with git hash-object --no-filters, which is how a file gets into the
object database with the line endings of the machine that added it.
