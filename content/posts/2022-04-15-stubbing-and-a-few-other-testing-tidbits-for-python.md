---
title: "Stubbing and few other testing tidbits for python"
description: "Stubbing and few other testing tidbits for python"
tags: [testing, python]
comments: true
share: true
cover_image: '/content/images/2016/7/decorators.png'
---

It's been sometime since I wrote some python, and ended doing a bit of testing for a couple of routines which I ended up implementing. This post is more about me just condensing those ideas for python and how to do it in python, but the ideas are also a carryover from my other testing experiences, while using other languages and how my ideas for testing have progressed over time comparing some testing which I had done in some projects some years back. You can find a couple of more posts under [https://www.tasdikrahman.com/blog/tag/testing/](https://www.tasdikrahman.com/blog/tag/testing/) where I have delved more into these topics.

As always, I will for sure look at this post at some point and notice improvements as my thoughts on testing progress and mature.

## What about non-deterministic tests

I picked this piece of code from an old project which I worked on long back in college for this bit, if you look at the following block

```python
# picked from here
# https://github.com/tasdikrahman/plino/blob/713ad80524bb4038cb08475b299b02cca3fe7feb/tests/test_plino_app_api_response.py#L37
    def test_api_spam_email_text(self):
        """
        Unit test to verify the 200 response code and the correct email_class
        returned by API when a spam email text is passed
        """
        payload = {
            'email_text': SPAM_TEXT
        }
        headers = {'content-type': 'application/json'}
        response = \
            requests.post(self.api_url, data=json.dumps(payload), headers=headers)
        r = json.loads(response.content)
        assert response.status_code == 200
        assert r['email_class'] == 'spam'
```

This for me, if I look at it now is more noise than signal, as the probability of it landing up in being classified to something which we are asserting it to, is probabilistic in nature, I would rather have this as a [test oracle](https://en.wikipedia.org/wiki/Test_oracle) for me to be able to know what is going on for this case at least.

Another thing to note here in this spec if that, there is a scope creep happening here, I am trying to do two things at the same time. One being, trying to test the route `api/v1/classify/`, and check for the response code for a successfull response and the second thing being, I am also trying to test the domain specific implementation of domain logic. Mixing these two don't really make sense at this point.

What I would have rather done at this point, is to inject the dependency of the domain specific logic to return the response for which I was testing the api response code for, making this spec deterministic and reducing the noise.

This would have also reduced the extra test behaviour which was being tested here in this case, which was unnecessary. Plus the underlying domain logic being tested was the classifier in this case, which would then be testing the 3rd party codebase itself, which is not required. What we would want to rather do is wrap our business logic around the responses which the 3rd party flow can give.

## Stubbing responses, in this case stubbing a method which receives STDIN

Will pluck out the irrelevant details from this spec I wrote for [`fileinput.input()`](https://docs.python.org/3/library/fileinput.html), the context was that a method was using fileinput.input() to read from STDIN, and we needed to test the original method, without actually waiting for STDIN in our test spec runner.

Here's a snippet describing changing the design of the implementation to prevent the call of fileinput.input() while the test run.

```python
# initial design
import fileinput

class IO:
    def read_stdin(self):
        """
        read_stdin will read the STDIN and process the data received

        :returns: list of sentences read line by line
        """
        lines = []
        cleaned_lines = []
        std_in = fileinput.input()
        for line in std_in:
            lines.append(line)
        # strip newlines
        cleaned_lines += [line.rstrip("\n") for line in lines]
        # remove empty strings
        return [x for x in cleaned_lines if x.strip()]
```

Now in our test spec, if we wanted to call this method and assert for list output which it returned, we would be stuck with the STDIN IO wait time here.

Rather adding an interface on top of this behaviour would help us further in stubbing that response out which we get from fileinput.input()

```python
import fileinput

class IO:
    def read_stdin(self):
        """
        read_stdin will read the STDIN and process the data received

        :returns: list of sentences read line by line
        """
        lines = []
        cleaned_lines = []
        std_in = self.get_stdin()
        for line in std_in:
            lines.append(line)
        # strip newlines
        cleaned_lines += [line.rstrip("\n") for line in lines]
        # remove empty strings
        return [x for x in cleaned_lines if x.strip()]

    @staticmethod
    def get_stdin():
        return fileinput.input()
```

Now we can stub this method in our spec, since we already knew the behaviour of the stubbed method and what it would give us, we added the response value for it for our spec, effectively replacing an actual call. It's interesting to see this in the decorator syntax provided and looks quite clean to read.

```python
import fileinput
from unittest import TestCase
from mock import patch

class TestReadStdin(TestCase):
    @patch.object(io.IO, "get_stdin")
    def test_read_stdin(self, stub_get_stdin):
        content = """I could not help it, but I began to feel suspicious of this. At any rate, I made up my mind that if it so turned out that we should sleep together, he must undress and get into bed before I did.

Supper over, the company went back to the bar-room, when, knowing not what else to do with myself, I resolved to spend the rest of the evening as a looker on."""
        want = [
            "I could not help it, but I began to feel suspicious of this. At any rate, I made up my mind that if it so turned out that we should sleep together, he must undress and get into bed before I did.",
            "Supper over, the company went back to the bar-room, when, knowing not what else to do with myself, I resolved to spend the rest of the evening as a looker on.",
        ]
        # create the temp file
        with TestFileContent(content) as valid_file:
            stub_get_stdin.return_value = fileinput.input(files=valid_file.filename)

            io_obj = io.IO()
            got = io_obj.read_stdin()

            self.assertEqual(want, got)
```

## Testing for STDOUT

It's of value to test out the STDOUT being received for certain cases. For example

```python
Class Printer:
    def print_this(self, key, value):
      print("{0} - {1}".format(" ".join(key), value))
```

spec for the same will look like

```python
from unittest import TestCase
import io
import unittest.mock
# import Printer

class TestPrinter(TestCase):
    @unittest.mock.patch("sys.stdout", new_callable=io.StringIO)
    def assert_stdout(self, test_input_a, test_input_b, expected_output, mock_stdout):
        print_obj = Printer()
        print_obj.print_this(test_input_a, test_input_b)
        self.assertEqual(mock_stdout.getvalue(), expected_output)

    def test_print_ranked(self):
        test_input_a = foo
        test_input_b = baz
        want = "foo - baz"

        self.assert_stdout(test_input_a, test_input_b, want)
```

The `mock_stdout` arg is passed automatically by the `unittest.mock.patch` decorator to the `assert_stdout` method.

## IO related to files

python already provides a great interface to creating temporary files and deleting them after their use case has been achieved, as compared to golang for example.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Is there a better way than to wrap the os.Remove(filename) with a defer, for a file created via io/ioutil in <a href="https://twitter.com/golang?ref_src=twsrc%5Etfw">@golang</a>?</p>&mdash; Tasdik Rahman (@tasdikrahman) <a href="https://twitter.com/tasdikrahman/status/1512697739196477440?ref_src=twsrc%5Etfw">April 9, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

The following block would add the content to the file straight up while creating the context block for the file.

```python
class TestFileContent:
    def __init__(self, content):
        self.file = tempfile.NamedTemporaryFile(mode="w", delete=False)

        with self.file as f:
            f.write(content)

    @property
    def filename(self):
        return self.file.name

    def __enter__(self):
        return self

    def __exit__(self, type, value, traceback):
        os.unlink(self.filename)
```

Now this can be simply used like this, whereas the name of the file can be plucked out via the filename attribute here.

```python
class TestReadFiles(TestCase):
    def test_read_files_read_single_file(self):
        content = """
Supper over, the company went back to the bar-room, when, knowing not what else to do with myself, I resolved to spend the rest of the evening as a looker on."""
        want = "test_output"        ]
        with TestFileContent(content) as valid_file:
            got = test_method([valid_file.filename])
            self.assertEqual(want, got)
```

If you wanted to create create multiple temporary files in the same context for simpler cleanup.

```python
        with TestFileContent(content_file_a) as file_a, TestFileContent(
            content_file_b
        ) as file_b:
          ...
          ...
```

Asserting for file not read errors

```python
    def test_read_files_file_not_found(self):
        with self.assertRaises(FileNotFoundError):
            io_obj = io.IO()
            io_obj.read_files(["non-existent-file.txt"])
```

In case you would like to test the behaviour when the user is not allowed to read the file content due to permission error

```python
    def test_read_files_file_read_permission_error(self):
        content = """foo"""
        with TestFileContent(content) as valid_file:
            io_obj = io.IO()
            # make file not readable to user
            os.chmod(valid_file.filename, 0o0230)
            with self.assertRaises(PermissionError):
                io_obj.read_files([valid_file.filename])
```

Will most likely add more references for myself here or in another post.

## References

- [https://stackoverflow.com/a/46307456](https://stackoverflow.com/a/46307456)
- [https://docs.python.org/3/library/fileinput.html](https://docs.python.org/3/library/fileinput.html)
- [https://stackoverflow.com/a/54053967](https://stackoverflow.com/a/54053967)
