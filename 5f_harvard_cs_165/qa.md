# Questions, and hopefully, Answers

1. Could compression in column-stores affect algorithm complexity of DB operations?
A. No. Complications would arise mostly from combinations of records. However, we keep records separate. Moreover, we are interested in compression-aware scanning. Compression is intended to affect storage. Compression algorithms include run-length encoding, bit-vector encoding, dictionary encoding, frame of reference encoding, differential encoding, and so on.
