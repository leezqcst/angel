# 同步模型
在分布式计算系统中，由于各个计算节点计算进度不可能完全一致，这就导致了在汇总结果时需要等待那些计算速度较慢的节点，即慢节点会拖慢整个计算任务的进度，浪费计算资源。如果计算任务需要的步骤比较少，这个问题可能不是很明显，但是如果计算步骤很多，尤其是像机器学习算法需要许多轮迭代的时候，这个问题会导致的时间和计算资源浪费就不能忽视了。

Angel构建在Yarn平台之上，物理机器本身就存在异构的状况，再加之与其他计算平台共享物理机器的资源，这就加剧了各计算节点（Worker）进度的不一致情况。考虑到机器学习的特殊性，我们可以适当放宽同步限制，没有必要每一轮都等待所有的计算节点完成计算，减少等待慢节点的时间从而减少整个任务的计算时间。

Angel提供了三个级别的同步协议： **BSP（Bulk Synchronous Parallel）**，**SSP（Stalness Synchronous Parallel）** 和 **ASP（Asynchronous Parallel）**， 它们的同步限制依次放宽。为了追求更快的计算速度，算法可以选择更宽松的同步协议。但是，当同步限制放宽之后可能导致收敛质量下降甚至任务不收敛的情况。因此，在实际应用中，需要用户根据实际情况调整同步协议以及相关的参数，以达到收敛性和计算速度的平衡。



# BSP
默认的同步协议。也是一般的分布式计算采用的同步协议，在每一轮迭代中都需要等待所有的Task计算完成。

 - 优点：适用范围广；每一轮迭代收敛质量高
 - 缺点：但是每一轮迭代都需要等待最慢的Task，整体任务计算时间长
 - 使用方式：默认的同步协议
 
 # SSP
允许一定程度的Task进度不一致，但这个不一致有一个上限，我们称之为**staleness**值，即最快的Task最多领先最慢的Task **staleness**轮迭代。

 - 优点：一定程度减少了Task之间的等待时间，计算速度较快
 - 缺点：每一轮迭代的收敛质量不如BSP，达到同样的收敛效果可能需要更多轮的迭代；适用性也不如BSP，部分算法不适用
 - 使用方式：配置参数`angel.staleness=N`，其中**N**为正整数
 
# ASP
Task之间完全不用相互等待。
 - 优点：消除了等待慢Task的时间，计算速度快
 - 缺点：适用性差，在一些情况下并不能保证收敛性
 - 使用方式：配置参数`angel.staleness=-1`


