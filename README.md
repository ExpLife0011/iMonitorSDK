English| [简体中文](./README-zh_CN.md)

## Introduction

iMonitorSDK is a development kit that provides system behavior monitoring for terminals and the cloud. Help industry applications such as security, management, and auditing can quickly implement necessary functions without worrying about underlying driver development, maintenance, and compatibility issues, allowing them to focus on business development.

iMonitorSDK also supports monitoring of processes, files, registry, network, system, etc., using standard and stable implementation methods, and also supports Windows (XP-Win11), Linux, and MacOS.

With iMonitorSDK, common terminal security functions such as self-protection, process interception, ransomware defense, active defense, and Internet behavior management can be realized at a very low cost.

### ✨ Core Functions

- Process, File, Registry Protection

- Process startup, module loading interception, module injection

- File interception and redirection

- Network firewall, traffic proxy, protocol analysis
- Rule engine, script support

### 📦 Applicable to the following products

- Endpoint Security Management System
- EDR
- HIPS
- Cloud Security
- Zero trust
- Internet Access Control

## 🔨 Quick start

Example 1: Process start interception

```c++
class MonitorCallback : public IMonitorCallback
{
public:
	void OnCallback(IMonitorMessage* Message) override
	{
		if (Message->GetType() != emMSGProcessCreate)
			return;

		cxMSGProcessCreate* msg = (cxMSGProcessCreate*)Message;

		//
		// Block the process of the process name cmd.exe from starting
		//

		if (msg->IsMatchPath(L"*\\cmd.exe"))
			msg->SetBlock();
	}
};

int main()
{
	MonitorManager manager;
	MonitorCallback callback;

	HRESULT hr = manager.Start(&callback);

	if (hr != S_OK) {
		printf("start failed = %08X\n", hr);
		return 0;
	}

	cxMSGUserSetMSGConfig config;
	config.Config[emMSGProcessCreate] = emMSGConfigSend;
	manager.InControl(config);

	WaitForExit("Block the process of the process name cmd.exe from starting");

	return 0;
}
```

Example 2: Self-protection

```c++
class MonitorCallback : public IMonitorCallback
{
public:
	void OnCallback(IMonitorMessage* Message) override
	{}
};

int main()
{
	MonitorManager manager;
	MonitorCallback callback;

	HRESULT hr = manager.Start(&callback);

	if (hr != S_OK) {
		printf("start failed = %08X\n", hr);
		return 0;
	}

	manager.InControl(cxMSGUserEnableProtect());

	{
		cxMSGUserAddProtectRule rule;
		rule.ProtectType = emProtectTypeProcessPath | emProtectTypeFilePath;
		wcsncpy(rule.Path, L"*\\notepad.exe", MONITOR_MAX_BUFFER);
		manager.InControl(rule);
	}

	{
		cxMSGUserAddProtectRule rule;
		rule.ProtectType = emProtectTypeFilePath;
		wcsncpy(rule.Path, L"*\\protect>", MONITOR_MAX_BUFFER);
		manager.InControl(rule);
	}

	{
		cxMSGUserAddProtectRule rule;
		rule.ProtectType = emProtectTypeRegPath;
		wcsncpy(rule.Path, L"*\\iMonitor>", MONITOR_MAX_BUFFER);
		manager.InControl(rule);
	}

	{
		cxMSGUserAddProtectRule rule;
		rule.ProtectType = emProtectTypeTrustProcess;
		wcsncpy(rule.Path, L"*taskkill*", MONITOR_MAX_BUFFER);
		manager.InControl(rule);
	}

	WaitForExit("SelfProtect");

	manager.InControl(cxMSGUserRemoveAllProtectRule());
	manager.InControl(cxMSGUserDisableProtect());

	return 0;
}
```

Example 3: sysmon

```c++
class MonitorCallback : public IMonitorCallback
{
public:
	void OnCallback(IMonitorMessage* msg) override
	{
		printf("%S ==> %S\n", msg->GetTypeName(), msg->GetFormatedString(emMSGFieldCurrentProcessPath));

		for (ULONG i = emMSGFieldCurrentProcessCommandline; i < msg->GetFieldCount(); i++) {
			printf("\t%30S : %-30S\n", msg->GetFieldName(i), msg->GetFormatedString(i));
		}
	}
};

int main()
{
	MonitorManager manager;
	MonitorCallback callback;

	HRESULT hr = manager.Start(&callback);

	if (hr != S_OK) {
		printf("start failed = %08X\n", hr);
		return 0;
	}

	cxMSGUserSetMSGConfig config;
	for (int i = 0; i < emMSGMax; i++) {
		config.Config[i] = emMSGConfigPost;
	}
	manager.InControl(config);

	WaitForExit("");

	return 0;
}
```

<img src="./doc/sysmon.gif" />

Example 4: Internet Access Control (based on network redirection, support https, refer to http_access_control example for details)

![](./doc/ac.png)

More examples can refer to the sample directory.

[For detailed instructions, please refer to the SDK documentation. ](./doc/README.md)

## License 

> Disclaimer:
>
> iMonitorSDK (hereinafter referred to as the SDK) is only licensed to be used by regular enterprise manufacturers. It is forbidden to be used in any illegal scenes such as endangering the safety of enterprises and individuals.
>
> This SDK comes with a kernel driver. Although it has undergone stable testing and long-term operation verification, compatibility problems will inevitably exist due to hardware and environmental reasons. Before using this SDK, please log in to the corresponding business environment system After fully testing, actually connect to use.
>
> The economic losses and legal issues caused by illegal authorization and illegal use have nothing to do with the SDK providing team.
>
> Before you use this SDK, it is deemed that you have known and complied with this disclaimer.

Functional differences of different licenses:

| Function description | Free License | Enterprise License | Enterprise Custom License |
| -------------------- | ------------ | ------------------ | ------------------------- |
| Process Monitoring | ✔ | ✔ | ✔ |
| File Monitoring | ✔ | ✔ | ✔ |
| Registry Monitoring | ✔ | ✔ | ✔ |
| Network Monitoring | ✔ | ✔ | ✔ |
| Self-protection | ✔ | ✔ | ✔ |
| Network Protocol Proxy | ✔ | ✔ | ✔ |
| Kernel object customization | | ✔ | ✔ |
| Configuration | | ✔ | ✔ |
| Rule Engine | | ✔ | ✔ |
| Javascript script support | | | ✔ |
| Linux support | | | ✔ |
| MacOS support | | | ✔ |
| Source code | | | ✔ |
| Service Support | Mail, GitHub | Mail, GitHub, WeChat, Remote Desktop | ✔ |

[ contact via email (iMonitor@qq.com) for a licence ](mailto://iMonitor@qq.com)

## Products using this SDK

- [iMonitor - Endpoint Behavior Analysis System](https://github.com/wecooperate/iMonitor)
- [iDefender - Endpoint Active Defense System](https://github.com/wecooperate/iDefender)

## About Us

Excellent people do professional things.

Wecooperate Technology is an enterprise dedicated to providing basic services and an integrated management platform for enterprise management, striving to become the entrance to enterprise management and promoting the standardization and digitization of enterprise management. Our goal is to reject involution and let everyone work and live better.

Our members are top talents from companies such as Kingsoft, 360, Tencent, etc., with deep technical skills. A number of core products are under development and require various talents and capital investment.

[Contact Us](mailto://iMonitor@qq.com)