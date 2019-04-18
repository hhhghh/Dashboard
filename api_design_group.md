# 小组搜索页面
1. ```getGroupByName(group.name)```: 根据组名来查找小组
2. ```getGroupByTag(tag)```: 根据标签来查找小组
3. ```applyToJoinGroup(user.id, group.name)```: 用户申请进组，判断小组的进组权限，直接进组/需要组长审核/禁止加入

# 我的小组页面
1. ```getGroupById(user.id)```: 根据用户id返回用户加入了的小组
2. ```getMembersByGroup(group.name)```: 根据组名来得到小组成员
3. ```isGroupLeader(group.name, user.id)```: 返回用户是否是该小组组长
4. ```disbandGroup(group.name, user.id)```: 判断该用户是否是小组的组长，是则解散小组，返回小组解散成功
5. ```removeMemberFromGroup(group.name, group_leader.id, member.id)```: 判断group_leader是否是这个小组的组长以及判断被踢成员是否在组里，若在，则踢出小组，返回踢出成功
6. ```transferGroup(group.name, group_leader.id, member.id)```: 判断group_leader是否是这个小组的组长以及判断被踢成员是否在组里，是则将小组转让，group_leader变成小组普通成员，member变成组长
7. ```withdrawGroup(group.name, user.id)```: 判断该用户是否是小组成员，如果是，则退出小组，返回成功
8. ```inviteUserToGroup(group.name, member.id, user.id)```: 判断小组的进组权限（是否需要验证）以及user是否存在，若不需要验证，则添加user到小组成员；若需要验证，则判断member是否为组长，是则添加，否则向组长发送验证通知
9. ```modifyGroupInformation(group, group_leader.id)```: 判断是否是组长，若是则修改小组信息

# 创建小组页面
1. ```isGroupNameExisted(group.name)```: 创建小组时用来判断组名是否已经被注册，若已经被注册，返回true
2. ```isUserExisted(user.id)```: 判断用户是否存在（创建小组时用来邀请成员，需要先判断一下用户是否存在，或者在下面的createGroup中判断也行）
3. ```createGroup(group, creator.id, vector<user.id>)```: 创建小组，creator为组长，vector中的user为小组成员（如果isUserExisted没有实现，这里需要逐个判断用户是否存在）

# 未完待续......