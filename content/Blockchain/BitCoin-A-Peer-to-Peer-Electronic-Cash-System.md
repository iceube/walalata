+++
date = '2025-07-23T18:12:47+08:00'
draft = false
title = 'BitCoin: A Peer to Peer Electronic Cash System'
+++

## 论文解读
**目标**：一个点对点的双花问题（double-spending）解决方案
**方法**：链式结构，hash，PoW，最长合法链，广播
**结果**：不可篡改
### 具体设计
**交易**：
一枚电子货币是一连串数字签名（Digital signatures）：每一位所有者对前一次交易和下一位拥有者的公钥（Public Key）签署一个随机散列的数字签名，并将这个签名附加在电子货币尾部，电子货币就发送给下一位所有者。收款人通过对签名进行验证（使用发布者的公钥），就能够验证该链条的所有者。（此时并不能防范双花问题）
<figure>
	<img src="https://cdn.nakamotoinstitute.org/img/library/bitcoin/transactions.svg">
	<center><figcaption>BAPPECS-交易流程图</figcaption></center>
</figure>
**时间戳服务器（Timestamp Server）：**（双花问题的解决）
每个时间戳应当将前一个时间戳纳入 hash，形成链条
<figure>
  <img src="https://cdn.nakamotoinstitute.org/img/library/bitcoin/timestamp-server.svg">
  <center><figcaption>BAPPECS-时间戳服务器</figcaption></center>
</figure>
**工作量证明（Proof of Work）：**
SHA-256
所需平均工作量与所需零位数呈指数关系，并且可以通过执行单个哈希来验证。
随机数（Nonce）使得区块满足；
一旦CPU耗费的工作量被用来满足工作量证明，该区块就不能在不重做工作的情况下被更改。由于后续的区块在其之后被链接，更改该区块的工作将包括重做其后的所有区块。（不可篡改）
<figure>
  <img src="https://cdn.nakamotoinstitute.org/img/library/bitcoin/proof-of-work.svg">
  <center><figcaption>BAPPECS-工作量证明</figcaption></center>
</figure>

PoW 同时解决投票问题，本质是one-CPU-one-vote
要修改过去的区块，攻击者必须重新完成该区块及其后所有区块的工作量证明，然后才能赶上并超过诚实节点的努力。（成功概率呈指数化递减）
工作量证明的难度（the proof-of-work difficulty）采用移动平均目标的方法决定，即令难度指向令每小时生成区块的速度为某一个预定的平均数。
**网络**：
1.	新交易向全节点广播
2.	每个节点将新交易纳入一个区块
3.	每个节点都尝试在自己的区块找到一个具有足够难度的工作量证明
4.	当找到一个 proof-of-work，就向全节点广播
5.	只有当区块中的交易都是合法且不曾存在过的，其它节点才会认同该区块的有效性（只要有足够多的节点承认就可以达成共识，对丢失信息有一定的容错能力）
6.	其它节点表示接受的方法是跟随在该区块的末尾
可能出现同时广播，各节点会在率先收到的区块基础上进行工作，僵局（tie）会在下一个工作量证明的时候被打破，其中一个会变更长，成为合法链。
**激励**：
1. 每个区块的第一笔交易（特殊交易）可以创造新币（由区块构建者）。
2. 交易费（transaction fees）：输入值与输出值的差额
**回收硬盘空间**：
Merkle tree：只有根（root）被纳入区块的 hash。老区块可以通过分枝拔除（stubbing）的方法被压缩。
<figure>
  <img src="https://cdn.nakamotoinstitute.org/img/library/bitcoin/reclaiming-disk-space.svg">
  <center><figcaption>BAPPECS-Merkle tree 示意图</figcaption></center>
</figure>
设定区块生成速率是 10 分钟 1 个。不含交易信息的区块头（Block header）大小仅有 80 字节。一年产生的数据位为4.2 MB。
**简化的支付确认（Simplified Payment Verification）：**
用户需要保留最长合法链的区块头副本（不断向网络发起询问），可以通过 merkle 的分支通向它被加上时间戳并纳入区块的那次交易。它不能自行验证交易的有效性，但通过追溯位置，可以看到交易被接受。
<figure>
  <img src="https://cdn.nakamotoinstitute.org/img/library/bitcoin/simplified-payment-verification.svg">
  <center><figcaption> BAPPECS-确认机制</figcaption></center>
</figure>
当诚实节点控制网络时，检验机制是可靠的。但是可能被计算力占优的攻击者攻击成功。
**价值的组合与分割（Combining and Splitting Value）**
<figure>
  <img src="https://cdn.nakamotoinstitute.org/img/library/bitcoin/combining-splitting-value.svg">
  <center><figcaption>BAPPECS-价值的组合与分割</figcaption></center>
</figure> 
一个（较大）或多个（较小）的输入
输出最多有两个：支付；找零。
隐私（Privacy）
保护隐私的方法：保持匿名公钥（每次交易可以都生成新地址，但并行输入仍然有被追踪的风险）
 <figure>
  <img src="https://cdn.nakamotoinstitute.org/img/library/bitcoin/privacy.svg">
  <center><figcaption>BAPPECS-隐私模型</figcaption></center>
</figure>
**计算:**
因为要验证交易的合法性，所以攻击者不能凭空创造价值或者掠夺不属于自己的货币，只能更改自己的交易信息，试图拿回自己刚付给别人的钱。
诚实链条和攻击链条之间的竞争：二叉随机游走（Binomial Random Walk）
成功事件：诚实链条延长一个区块，领先 + 1
失败事件：攻击者链条延长一个区块，差距 - 1
攻击者填补差距取代最长链可以近似地看作赌徒破产问题（Gambler's problem）。假定一个赌徒拥有无限的透支信用，然后开始进行潜在次数为无穷的赌博，试图填补上自己的亏空。我们可以计算赌徒填补上亏空的概率，也就是该攻击者赶上诚实链条的概率。
<math xmlns="http://www.w3.org/1998/Math/MathML" display="block">
  <mtable displaystyle="true" columnalign="right left" columnspacing="0em" rowspacing="3pt">
    <mtr>
      <mtd>
        <mi>p</mi>
      </mtd>
      <mtd>
        <mi></mi>
        <mo>=</mo>
        <mtext>probability an honest node finds the next block</mtext>
      </mtd>
    </mtr>
    <mtr>
      <mtd>
        <mi>q</mi>
      </mtd>
      <mtd>
        <mi></mi>
        <mo>=</mo>
        <mtext>probability the attacker finds the next block</mtext>
      </mtd>
    </mtr>
    <mtr>
      <mtd>
        <msub>
          <mi>q</mi>
          <mi>z</mi>
        </msub>
      </mtd>
      <mtd>
        <mi></mi>
        <mo>=</mo>
        <mrow>
          <mtext>probability the attacker will ever catch up from&#xA0;</mtext>
          <mrow data-mjx-texclass="ORD">
            <mi>z</mi>
          </mrow>
          <mtext>&#xA0;blocks behind</mtext>
        </mrow>
      </mtd>
    </mtr>
  </mtable>
</math>
计算概率：
<math xmlns="http://www.w3.org/1998/Math/MathML" display="block">
  <mstyle mathsize="1.2em">
    <msub>
      <mi>q</mi>
      <mi>z</mi>
    </msub>
    <mo>=</mo>
    <mrow data-mjx-texclass="INNER">
      <mo data-mjx-texclass="OPEN">{</mo>
      <mtable columnspacing="1em" rowspacing="4pt">
        <mtr>
          <mtd>
            <mn>1</mn>
          </mtd>
          <mtd>
            <mtext>if</mtext>
            <mstyle scriptlevel="0">
              <mspace width="0.278em"></mspace>
            </mstyle>
            <mi>p</mi>
            <mo>&#x2264;</mo>
            <mi>q</mi>
          </mtd>
        </mtr>
        <mtr>
          <mtd>
            <mo stretchy="false">(</mo>
            <mi>q</mi>
            <mrow data-mjx-texclass="ORD">
              <mo>/</mo>
            </mrow>
            <mi>p</mi>
            <msup>
              <mo stretchy="false">)</mo>
              <mi>z</mi>
            </msup>
          </mtd>
          <mtd>
            <mtext>if</mtext>
            <mstyle scriptlevel="0">
              <mspace width="0.278em"></mspace>
            </mstyle>
            <mi>p</mi>
            <mo>&gt;</mo>
            <mi>q</mi>
          </mtd>
        </mtr>
      </mtable>
      <mo data-mjx-texclass="CLOSE">}</mo>
    </mrow>
  </mstyle>
</math>
动态密钥对（Dynamic Key Pairs）防止预计算攻击（Precomputation Attack）
攻击者的潜在进展是一个泊松分布，期望值为
<math xmlns="http://www.w3.org/1998/Math/MathML" display="block">
  <mi>&#x3BB;</mi>
  <mo>=</mo>
  <mi>z</mi>
  <mfrac>
    <mi>q</mi>
    <mi>p</mi>
  </mfrac>
</math>
此时的总概率（k=1，2，···）为：
<math xmlns="http://www.w3.org/1998/Math/MathML" display="block">
  <munderover>
    <mo data-mjx-texclass="OP">&#x2211;</mo>
    <mrow data-mjx-texclass="ORD">
      <mi>k</mi>
      <mo>=</mo>
      <mn>0</mn>
    </mrow>
    <mrow data-mjx-texclass="ORD">
      <mi mathvariant="normal">&#x221E;</mi>
    </mrow>
  </munderover>
  <mfrac>
    <mrow>
      <msup>
        <mi>&#x3BB;</mi>
        <mi>k</mi>
      </msup>
      <msup>
        <mi>e</mi>
        <mrow data-mjx-texclass="ORD">
          <mo>&#x2212;</mo>
          <mi>&#x3BB;</mi>
        </mrow>
      </msup>
    </mrow>
    <mrow>
      <mi>k</mi>
      <mo>!</mo>
    </mrow>
  </mfrac>
  <mo>&#x22C5;</mo>
  <mrow data-mjx-texclass="INNER">
    <mo data-mjx-texclass="OPEN">{</mo>
    <mtable columnspacing="1em" rowspacing="4pt">
      <mtr>
        <mtd>
          <mo stretchy="false">(</mo>
          <mi>q</mi>
          <mrow data-mjx-texclass="ORD">
            <mo>/</mo>
          </mrow>
          <mi>p</mi>
          <msup>
            <mo stretchy="false">)</mo>
            <mrow data-mjx-texclass="ORD">
              <mo stretchy="false">(</mo>
              <mi>z</mi>
              <mo>&#x2212;</mo>
              <mi>k</mi>
              <mo stretchy="false">)</mo>
            </mrow>
          </msup>
        </mtd>
        <mtd>
          <mtext>if</mtext>
          <mstyle scriptlevel="0">
            <mspace width="0.278em"></mspace>
          </mstyle>
          <mi>k</mi>
          <mo>&#x2264;</mo>
          <mi>z</mi>
        </mtd>
      </mtr>
      <mtr>
        <mtd>
          <mn>1</mn>
        </mtd>
        <mtd>
          <mtext>if</mtext>
          <mstyle scriptlevel="0">
            <mspace width="0.278em"></mspace>
          </mstyle>
          <mi>k</mi>
          <mo>&gt;</mo>
          <mi>z</mi>
        </mtd>
      </mtr>
    </mtable>
    <mo data-mjx-texclass="CLOSE">}</mo>
  </mrow>
</math>
重新排列为：
<math xmlns="http://www.w3.org/1998/Math/MathML" display="block">
  <mn>1</mn>
  <mo>&#x2212;</mo>
  <munderover>
    <mo data-mjx-texclass="OP">&#x2211;</mo>
    <mrow data-mjx-texclass="ORD">
      <mi>k</mi>
      <mo>=</mo>
      <mn>0</mn>
    </mrow>
    <mrow data-mjx-texclass="ORD">
      <mi>z</mi>
    </mrow>
  </munderover>
  <mfrac>
    <mrow>
      <msup>
        <mi>&#x3BB;</mi>
        <mi>k</mi>
      </msup>
      <msup>
        <mi>e</mi>
        <mrow data-mjx-texclass="ORD">
          <mo>&#x2212;</mo>
          <mi>&#x3BB;</mi>
        </mrow>
      </msup>
    </mrow>
    <mrow>
      <mi>k</mi>
      <mo>!</mo>
    </mrow>
  </mfrac>
  <mrow data-mjx-texclass="INNER">
    <mo data-mjx-texclass="OPEN">(</mo>
    <mn>1</mn>
    <mo>&#x2212;</mo>
    <mo stretchy="false">(</mo>
    <mi>q</mi>
    <mrow data-mjx-texclass="ORD">
      <mo>/</mo>
    </mrow>
    <mi>p</mi>
    <msup>
      <mo stretchy="false">)</mo>
      <mrow data-mjx-texclass="ORD">
        <mo stretchy="false">(</mo>
        <mi>z</mi>
        <mo>&#x2212;</mo>
        <mi>k</mi>
        <mo stretchy="false">)</mo>
      </mrow>
    </msup>
    <mo data-mjx-texclass="CLOSE">)</mo>
  </mrow>
</math>
C 语言格式为
~~~ C
#include <math.h>
double AttackerSuccessProbability(double q, int z)
{
	double p = 1.0 - q;
	double lambda = z * (q / p);
	double sum = 1.0;
	int i, k;
	for (k = 0; k <= z; k++)
	{
		double poisson = exp(-lambda);
		for (i = 1; i <= k; i++)
			poisson *= lambda / i;
		sum -= poisson * (1 - pow(q / p, z - k));
	}
	return sum;
}
~~~
比特币规定：6 个区块进行确认





