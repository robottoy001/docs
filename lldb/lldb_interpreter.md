# LLDB commnd line
## Interpreter
用来解析输入的命令，转发调用适当命令对象。命令对应的处理对象会在启动的时候注册好，并通过直接或间接（正则匹配）方式找到具体的注册表项，根据表项，找到对应的执行command对象，执行command对象的Execute函数（经典的Command模式）
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
### 命令注册
- 注册builtin命令使用如下宏
```C++
#define REGISTER_COMMAND_OBJECT(NAME, CLASS)                                   \
  m_command_dict[NAME] = std::make_shared<CLASS>(*this);
```
- 在LoadCommandDictionary中注册并初始化
```C++
void CommandInterpreter::LoadCommandDictionary() {
  LLDB_SCOPED_TIMER();

  REGISTER_COMMAND_OBJECT("apropos", CommandObjectApropos);
  REGISTER_COMMAND_OBJECT("breakpoint", CommandObjectMultiwordBreakpoint);
  REGISTER_COMMAND_OBJECT("command", CommandObjectMultiwordCommands);
  REGISTER_COMMAND_OBJECT("disassemble", CommandObjectDisassemble);
  REGISTER_COMMAND_OBJECT("expression", CommandObjectExpression);
  REGISTER_COMMAND_OBJECT("frame", CommandObjectMultiwordFrame);
  REGISTER_COMMAND_OBJECT("gui", CommandObjectGUI);
  REGISTER_COMMAND_OBJECT("help", CommandObjectHelp);
  REGISTER_COMMAND_OBJECT("log", CommandObjectLog);
  REGISTER_COMMAND_OBJECT("memory", CommandObjectMemory);
  REGISTER_COMMAND_OBJECT("platform", CommandObjectPlatform);
  REGISTER_COMMAND_OBJECT("plugin", CommandObjectPlugin);
  REGISTER_COMMAND_OBJECT("process", CommandObjectMultiwordProcess);
.....
}
```
### Alias command
给命令起个别名会比较方便用户使用，attach命令实际是“_regexp-attach”的别名,在解释器初始化的时候会添加
```C++
void CommandInterpreter::Initialize() {
....
  // Set up some initial aliases.
  CommandObjectSP cmd_obj_sp = GetCommandSPExact("quit");
  if (cmd_obj_sp) {
    AddAlias("q", cmd_obj_sp);
    AddAlias("exit", cmd_obj_sp);
  }

  cmd_obj_sp = GetCommandSPExact("_regexp-attach");
  if (cmd_obj_sp)
    AddAlias("attach", cmd_obj_sp)->SetSyntax(cmd_obj_sp->GetSyntax());
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

### Command
command 对象的实现解耦的很好，可以直接看source/Command目录里面所有命令。其中，有些命令还子命令，那么会继承CommandObjectMultiword，比如CommandObjectMultiwordProcess。Command模式比较简单，不在赘述

### Refer file
- CommandInterpreter.h
- CommandInterpreter.cpp
- CommandObjectProcess.cpp