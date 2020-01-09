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
            

    



