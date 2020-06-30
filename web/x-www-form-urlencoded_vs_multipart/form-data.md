from https://stackoverflow.com/questions/4007969/application-x-www-form-urlencoded-or-multipart-form-data, EML

READ AT LEAST THE FIRST PARA HERE!

I know this is 3 years too late, but Matt's (accepted) answer is incomplete and will eventually get you into trouble. The key here is that, if you choose to use multipart/form-data, the boundary must not appear in the file data that the server eventually receives.

This is not a problem for application/x-www-form-urlencoded, because there is no boundary. x-www-form-urlencoded can also always handle binary data, by the simple expedient of turning one arbitrary byte into three 7BIT bytes. Inefficient, but it works (and note that the comment about not being able to send filenames as well as binary data is incorrect; you just send it as another key/value pair).

The problem with multipart/form-data is that the boundary separator must not be present in the file data (see RFC 2388; section 5.2 also includes a rather lame excuse for not having a proper aggregate MIME type that avoids this problem).

So, at first sight, multipart/form-data is of no value whatsoever in any file upload, binary or otherwise. If you don't choose your boundary correctly, then you will eventually have a problem, whether you're sending plain text or raw binary - the server will find a boundary in the wrong place, and your file will be truncated, or the POST will fail.

The key is to choose an encoding and a boundary such that your selected boundary characters cannot appear in the encoded output. One simple solution is to use base64 (do not use raw binary). In base64 3 arbitrary bytes are encoded into four 7-bit characters, where the output character set is [A-Za-z0-9+/=] (i.e. alphanumerics, '+', '/' or '='). = is a special case, and may only appear at the end of the encoded output, as a single = or a double ==. Now, choose your boundary as a 7-bit ASCII string which cannot appear in base64 output. Many choices you see on the net fail this test - the MDN forms docs, for example, use "blob" as a boundary when sending binary data - not good. However, something like "!blob!" will never appear in base64 output.