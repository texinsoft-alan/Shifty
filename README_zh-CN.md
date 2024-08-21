Shifty
======
适用于 Arduino 的灵活 74HC595 管理器
--------------------------------------

English description please [click here](./README.md).

Arduino 的 Shifty 库是一种非常灵活的管理 74HC595 移位寄存器的方法。 它允许您像“`digitalWrite`”一样写入单个输出，允许您以串联的方式将移位寄存器连接在一起，并且，如果您按照本文档中的说明进行接线，则允许您将移位寄存器用于输入和输出引脚，只需使用一个额外的引脚。 这使得它非常适合与 ATTiny 一起使用，尽管它确实占用了设备上的一些空间。

### 灵感

这首先受到 ATTiny85 的启发，这是一款非常棒的芯片，如果有限制的话。 使用 ATTiny85 做任何花哨的事情几乎都需要移位寄存器。 然而，移位寄存器几乎占用了所有引脚，使得输入变得困难。 然后，我偶然发现了 Kevin Darrah 关于如何使用输出移位寄存器读取输入的视频。 问题是代码非常晦涩难懂。 因此，我决定编写一个库，将这些想法封装成易于使用的形式。

### 基本用法

如果您有一个单移位寄存器，其数据位于引脚 11 上，时钟位于引脚 12 上，并且锁存器位于引脚 8 上，则在寄存器上闪烁 Q3 的简单程序将如下所示：

    Shifty myreg;

    void setup() {
      myreg.setBitCount(8);
      myreg.setPins(11, 12, 8);
    }

    void loop() {
      myreg.writeBit(3, HIGH);
      delay(500);
      myreg.writeBit(3, LOW);
      delay(500);
    }

### 批处理模式

为了使移位寄存器上的读/写位更简单，Shifty 会缓冲您的输出，然后在每次设置位时重写所有输出。 但是，如果您想加快此速度，可以将多个位集包装到单个批处理中：

    void loop() {
      myreg.batchWriteBegin();
      myreg.writeBit(3, HIGH);
      myreg.writeBit(4, LOW);
      myreg.writeBit(5, HIGH);
      myreg.writeBit(8, LOW);
      myreg.batchWriteEnd();
    }

这将设置 Q3、Q4、Q5 和 Q8 上的位，但在您发出 `batchWriteEnd()` 命令之前不会将结果推出。

### 将移位寄存器串联在一起

此外，如果以串联的方式将两个 74HC595 连接在一起，则可以使用相同的库，只需将其设置为 16 位而不是 8 位：

    Shifty myreg;

    void setup() {
      myreg.setBitCount(16);
      myreg.setPins(11, 12, 8);
    }

您可以根据需要将任意数量的 74HC595 连接在一起，除非出现电气问题。 从技术上讲，库将您限制为 16 个，但这很容易更改。 但是，随着移位寄存器的增加，推出结果所需的时间会增加。 对于每个 `batchWriteEnd()`（如果您不在批处理中，则为每个 writeBit），它必须写入每个移位寄存器位。

### 输入引脚

使用输出移位寄存器进行输入的灵感来自于 Kevin Darrah 的这个 YouTube 视频：

http://www.youtube.com/watch?v=nXl4fb_LbcI

请注意，输入仍在积极开发中，目前可能正常运行，也可能不起作用。

以下是设置 API 以使用的方式。 首先，这需要使用单个输入引脚来接收输入反馈。 因此，setPins（） 需要一个额外的参数 - 最后一个数字将是用于输入通道的引脚。 此外，您计划用于输入的每个位都需要用 `bitMode()` 进行标记。

    Shifty myreg;
    void setup() {
      myreg.setBitCount(8);
      myreg.setPins(11, 12, 8, 9);
      myreg.bitMode(4, INPUT);
      myreg.bitMode(6, INPUT);
    }

这将在芯片上设置 Q4 和 Q6 用作输入位。 请注意，寄存器本身不支持输入，相反，您必须按照下面（或视频）所述进行接线，以将其用作输入位。 简而言之，发生的情况是“输入”位设置为 HIGH ，然后我们检查是否可以读取输入引脚上的 HIGH 信号，以查看该特定引脚的状态。

读取引脚的速度很慢，但 ATTiny 足够快，通常无关紧要。

因此，要读取引脚，请执行以下操作：

    void loop() {
      if(myreg.readBit(4) == HIGH) {
        myreg.writeBit(5, HIGH);
      } else { 
        myreg.writeBit(5, LOW);
      }
    }
