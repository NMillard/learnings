# Streams
Simply put, a stream is a sequence of bytes that you can read from and write to a backing store.  
  
You'll typically work with stream readers and writers. These are classes that knows how to interact with the underlying stream.
Readers and writers take care of conversion/encoding, this allows you to work with higher level constructs such as int, string, classes etc.
instead of working directly with bytes.  
  
