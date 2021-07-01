


|os and os.path |	pathlib |
| -------------| --------- |
|os.path.abspath |	Path.resolve |
|os.chmod |	Path.chmod |
|os.mkdir |	Path.mkdir |
|os.rename |	Path.rename |
|os.replace |	Path.replace |
|os.rmdir |	Path.rmdir |
|os.remove, os.unlink |	Path.unlink |
|os.getcwd |	Path.cwd |
|os.path.exists	 |Path.exists |
|os.path.expanduser	 |Path.expanduser and Path.home |
|os.path.isdir |	Path.is_dir |
|os.path.isfile |	Path.is_file |
|os.path.islink |	Path.is_symlink |
|os.stat |	Path.stat, Path.owner, Path.group |
|os.path.isabs |	PurePath.is_absolute |
|os.path.join |	PurePath.joinpath |
|os.path.basename |	PurePath.name |
|os.path.dirname |	PurePath.parent |
|os.path.samefile |	Path.samefile |
|os.path.splitext |	PurePath.suffix |


我这里就举一个例子：
```
    def load(self, log_path):
        self.log_path = Path(log_path).resolve(True)
        print(self.log_path)

        if self.log_path.is_file():
            try:
                file_name = self.log_path.name
                print("File_name:\n", file_name)
                data_begain = 0
                with self.log_path.open() as f:
                    for line in f:
                        data = line.split()
```

或者

```
    def load(self, log_path):
        self.log_path = Path(log_path).resolve(True)
        self.raw = self.load_raw()


    def load_raw(self):
        print(f"loading raw logs from \"{self.log_path}\"")

        logs = []

        for entry in self.log_path.glob("*.txt"):
            label = -1
            print("Entry:\n", entry)
            if entry.is_file():
                try:
                    file_name = entry.name
                    print("File_name:\n", file_name)

                    if fd_log in file_name:
                        label = 1
                    # print("label = ", label)

                    text = entry.read_text()
                    data = re.findall(r'BEGIN_UUT.*?Dmax = (.*?)\nSmax = (.*?)\n(.*?)Min:.*?Max:.*?Average:.*?END_UUT',
                                      text, re.DOTALL)
                    # print("Data", data)

                    for raw_data in data:
```


please refer:
https://docs.python.org/zh-cn/3/library/pathlib.html
