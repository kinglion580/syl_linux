# question
## Lookup files and output information as required
### introduce

The challenges you need to find the `/etc` directory to meet the needs of the file, and then the output information for simple processing.

All the `.conf` at the end of the first to use the find command to find the file below `/etc`, and will find the command stderr redirect to the `/home/shiyanlou/error.txt` file, the stdout output sort of content (default alphabetical order) after the redirect to the `/home/shiyanlou/conflist.txt` file.

Click to submit the result after the completion of the above task.

### target

1. The find command does not use sudo, and there will be error information output. We need to save the `error information` to the `/home/shiyanlou/error.txt` file


2. Every line in `/home/shiyanlou/conflist.txt` file is an absolute path of a `.conf` file, and it is sorted according to the alphabetical order of the absolute path.

### Hint

Lookup, pipe, sort, redirection

### Knowledge point

- Linux file lookup operation
- Find command
- Pipe character
 Sort sorting
- Redirecting stdout and stderr

# keypoint
## standard error output to file
> `find /etc -name "*.conf" 2>error.txt`
> - `-name`
> - `2>error.txt`

# references
[standard output,standard error output](http://blog.sina.com.cn/s/blog_4824ac470100tous.html)
