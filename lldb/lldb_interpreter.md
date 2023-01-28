# LLDB commnd line
## Interpreter
用来解析输入的命令，转发调用适当命令对象
### 命令路由
- 所有builtin命令会注册在解释器的m_command_dict中，便于查找处理函数
- 所有别名命令放在m_alais_dict
- 所有用户自定义的命令放在m_usr_dict
  
查找命令顺序，按照builtin->alais->user顺序查找
```C++
  CommandObject::CommandMap m_command_dict; // Stores basic built-in commands
                                            // (they cannot be deleted, removed
                                            // or overwritten).
  CommandObject::CommandMap
      m_alias_dict; // Stores user aliases/abbreviations for commands
  CommandObject::CommandMap m_user_dict; // Stores user-defined commands
```
### Regex command
有些命令可以简写，比如"process attach"，可以直接使用attach来用，在解释器内部会注册一个命令别名为“attach”，实际命令名称为“_regexp-attach”,这个命令会初始化Regex表达式，匹配后面的参数

```C++
  std::unique_ptr<CommandObjectRegexCommand> attach_regex_cmd_up(
      new CommandObjectRegexCommand(
          *this, "_regexp-attach", "Attach to process by ID or name.",
          "_regexp-attach <pid> | <process-name>", 2, 0, false));
  if (attach_regex_cmd_up) {
    if (attach_regex_cmd_up->AddRegexCommand("^([0-9]+)[[:space:]]*$",
                                             "process attach --pid %1") &&
        attach_regex_cmd_up->AddRegexCommand(
            "^(-.*|.* -.*)$", "process attach %1") && // Any options that are
                                                      // specified get passed to
                                                      // 'process attach'
        attach_regex_cmd_up->AddRegexCommand("^(.+)$",
                                             "process attach --name '%1'") &&
        attach_regex_cmd_up->AddRegexCommand("^$", "process attach")) {
      CommandObjectSP attach_regex_cmd_sp(attach_regex_cmd_up.release());
      m_command_dict[std::string(attach_regex_cmd_sp->GetCommandName())] =
          attach_regex_cmd_sp;
    }
  }
```