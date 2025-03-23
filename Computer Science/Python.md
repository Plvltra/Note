#### 
- 深入理解 Python 虚拟机: https://nanguage.gitbook.io/inside-python-vm-cn
- CPython函数接口分为两种引用: reference stealing and reference borrowing
``` cpp
PyObject* PyList = PyList_New(SetupCommands.Num());  
for (int32 i = 0; i < SetupCommands.Num(); i++)  
{  
    PyObject* PyString = PyUnicode_FromString(TCHAR_TO_UTF8(*SetupCommands[i]));  
    PyList_SetItem(PyList, i, PyString);  
}  
```
例如上例，PyObject* PyString被创建时引用计数为1。PyList_SetItem接口是reference stealing，不增加PyString的引用计数只是steal PyString的引用。当Py_XDECREF(PyList)后，PyString的引用也会归零。

- Python创建虚拟环境
```
python -m venv {venv_folder} # 创建虚拟环境
pip install -r requirements.txt # 根据requirement.txt在虚拟环境中安装依赖
source {venv_folder}/Scripts/activate # 进入虚拟环境
deactivate # 退出虚拟环境
```
- C++调用Python示例
``` cpp
typedef TPanguPyPtr<PyObject> FPanguPyObjectPtr;

/** Copy a pointer */
TPanguPyPtr(const TPanguPyPtr& InOther)  
    : Ptr(InOther.Ptr)  
{  
    Py_XINCREF(Ptr);  
}
/** Release our reference to this object */  
~TPanguPyPtr()  
{  
    Py_XDECREF(Ptr);  
}

/** Create a pointer from the given object, leaving its reference count alone */  
TPanguPyPtr StealReference(TPythonType* InPtr)

void UPanguPyExtender::SendMsgToUsers(const FString& Msg, const TArray<FString>& UserList)
{
	TArray<FString> FixedUserList;
	for (const FString& Username : UserList)
	{
		const UPanguProjectSettings* PanguProjectSettings = GetDefault<UPanguProjectSettings>();
		FString FixedUsername = PanguProjectSettings->TryGetUserEmailFromPopoAccountTable(Username);
		if (FixedUsername.IsEmpty())
		{
			FixedUsername = Username;
			FixedUserList.Emplace(FixedUsername);
		}
	}

	FPanguPyScopedGIL GIL;
	FPanguPyObjectPtr PyPanguPopoUtilModule = FPanguPyObjectPtr::StealReference(PyImport_ImportModule(PANGU_SCRIPT_POPOUTIL_MODULE));
	if (PyPanguPopoUtilModule)
	{
		if (FPanguPyObjectPtr PySendFunc = FPanguPyObjectPtr::StealReference(PyObject_GetAttrString(PyPanguPopoUtilModule, PANGU_SCRIPT_SEND_MSG_TO_USERS)))
		{
			FPanguPyObjectPtr PyMsg = FPanguPyObjectPtr::StealReference(PyUnicode_FromString(TCHAR_TO_UTF8(*Msg)));
			FPanguPyObjectPtr PyList = FPanguPyObjectPtr::StealReference(PyList_New(UserList.Num()));
			for (int32 i = 0; i < UserList.Num(); i++)
			{
				PyObject* PyString = PyUnicode_FromString(TCHAR_TO_UTF8(*UserList[i]));
				PyList_SetItem(PyList, i, PyString);
			}
			FPanguPyObjectPtr PyArgs = FPanguPyObjectPtr::StealReference(Py_BuildValue("(OO)", PyMsg.Get(), PyList.Get()));
			FPanguPyObjectPtr Result = FPanguPyObjectPtr::StealReference(PyObject_CallObject(PySendFunc, PyArgs));
			if (!Result)
			{
				TryPrintPyError();
				return;
			}
			return;
		}
	}
	return;
}
```

#### 踩坑
- pip 或 meson出问题
```bash
# https://github.com/mesonbuild/meson/issues/7258
sudo -i
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python3 get-pip.py
python3 -m pip install meson
```

#### 技巧
- 使用raw string避免正则表达式写法过长