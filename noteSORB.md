# note of SORB
by Kang Yongxin 2020 0108


代码流程

agent 定义在 UvfAgent中 。。。

有两个环境，一个是用来训练，一个用来eval 


train_eval 函数：
    输入是： agent=UvfAgent()
            tf_env
            eval_tf_env(?? 这个eval是什么含义)
    输出： train loss

    循环之前设置了一些初始化，包括：
        replay_buffer；
        eval_metrics；
        eval_policy（用actor policy网络初始化）；
        collect_policy（Actor Policy with Ornstein Uhlenbeck (OU) exploration noise，来源于OUNoisePolicy类，它类似于探索的网络）；
        collect_driver(未知用途)
    主循环是 for _ in tqdm.tnrange(num_iterations):
        collect_driver 得到 policy state
        重点关注图的构建和轨迹生成部分：

        get_dist_to_goal
            <-get_state_values(next_time_step)
                <-_get_expected_q_values(next_time_steps,action)#各个动作
                    <-_get_critic_output(?,next_states,acts)
                    用评价网络根据状态和动作给出一个评价值

        train 的过程在agent中完成
            trajectory[time_steps, policys, next_steps]
            time_step [reward, discount, obs]
            为啥把goal放到obs中，而不是放在reward中
            这里的图是用一个向量组表示的
            

visualize rollouts 是画图的

在replay buffer 中产生随机数据 rb_vec
pdist = agent._get_pairrwise_dist()  pdist.shape (3, 1000, 1000)
    pseudo_next_time_steps 找到下一个状态
    _get_dist_to_goal（）
        ._get_state_values（）
            _get_expected_q_values（）
                q_values_list = self._get_critic_output（）
                    q_values, _ = critic_net(obs ,act) 
算完之后怎么用了呢？

在search_policy.py 中用 pdist来创建图 self._buildgraph		g = nx.DiGraph()
		pdist_combined = np.max(pdist, axis=0)
		for i, s_i in enumerate(rb_vec):
			for j, s_j in enumerate(rb_vec):
				length = pdist_combined[i, j]
				if length < self._agent._max_search_steps:
					g.add_edge(i, j, weight=length)
		return g

所以我们的目标是要把整个过程建立图，然后对图进行优化，优化之后再在图中进行搜索。

search_policy._get_path 是用networkx.shortest_path 来计算最短路径的 ， 得到waypoint_vec, wypt_to_goal_dist[1:]，记录路径中的点,

search_policy._action 把这些点设置为goal,放入time_step中去。返回self._agent.policy.action(time_step, policy_state, seed)

在训练过程中是有dist计算的。训练时候会对下一个状态有影响，而且是有goal 参与到其中的


这是个无向图，无向图如何聚类？


总结：
    给agent的输入是两个，一个是obs,一个是goal
    在训练的过程中不动obs,只通过搜索算法不断改进goal





