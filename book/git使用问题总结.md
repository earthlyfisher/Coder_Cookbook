#������ȥ��δ��
##��Խ����ȥ
�������ύ�����ɰ汾������˵���ǰĳһ�汾ʱ�����ȣ�Git����֪����ǰ�汾���ĸ��汾����Git�У���HEAD��ʾ��ǰ�汾��Ҳ�������µ��ύ3628164...882e1e0����Ȼ���id��һ��һ��������һ���汾����HEAD^������һ���汾����HEAD^^����Ȼ����100���汾д100��^�Ƚ�������������������д��HEAD~100��
�������ύ�İ汾��Ϣ��
```
$ git log --pretty=oneline
97ce1a41b9ea6192254da48c634263e38910c638 append
a00bd1c06b77755a57ddb52eecedf1b1b2e7d577 singleton
394cd290e2a9392b4b5ee669bfe2805b1c92277e websocket
79c063b10583a7072b703056ea157b54fc08ef81 Merge branch 'master' of github.com:earthlyfisher/java-book
ed915264579ee3a7060cc141b9b31d06fec562b1 websocket
```
*`--pretty=oneline`���Խ�log��Ϣ��ʾ��һ��*

���ص���һ���汾��
```
$ git reset --hard HEAD^
HEAD is now at a00bd1c singleton
```
��Ȼ����ͨ��id����ʽȥ���ˣ�����HEAD^��commit idΪa00bd1c06b77...,���Կ���ͨ�����淽ʽ:
```
git reset --hard HEAD^
HEAD is now at a00bd1c a00bd1c06b77
```
##��Խ��δ��
�������ϵĻ��˲���ʹ������ͨ��log��־û��׷��ǰһ��commit id�������а취�ģ�����ͨ��`reflog`׷�ٲ�����־:
```
$ git reflog
97ce1a4 HEAD@{0}: reset: moving to 97ce1a41
a00bd1c HEAD@{1}: reset: moving to HEAD^
97ce1a4 HEAD@{2}: commit: append
a00bd1c HEAD@{3}: commit: singleton
```
���Կ���`append`��β���`commit id`��¼�����Կ���ͨ��
```
$ git reset --hard 97ce1a41
HEAD is now at 97ce1a4 append
```
��Խ��δ��
##summary
* `HEAD`ָ��İ汾���ǵ�ǰ�汾����ˣ�Git���������ڰ汾����ʷ֮�䴩��ʹ������`git reset --hard commit_id`
* ����ǰ����`git log`���Բ鿴�ύ��ʷ���Ա�ȷ��Ҫ���˵��ĸ��汾
* Ҫ�ط�δ������`git reflog`�鿴������ʷ���Ա�ȷ��Ҫ�ص�δ�����ĸ��汾

#�����޸�
����������ļ������ǹ����־��������⣬��ص���θ���ǰ��״̬������ͨ��
```
$ git checkout -- 13.txt
```
����`git checkout -- readme.txt`��˼���ǣ���`readme.txt`�ļ��ڹ��������޸�ȫ�����������������������

һ����`readme.txt`���޸ĺ�û�б��ŵ��ݴ��������ڣ������޸ľͻص��Ͱ汾��һģһ����״̬��

һ����`readme.txt`�Ѿ���ӵ��ݴ������������޸ģ����ڣ������޸ľͻص���ӵ��ݴ������״̬��

��֮������������ļ��ص����һ��`git commit`��`git add`ʱ��״̬��

*`git checkout -- file`�����е�--����Ҫ��û��--���ͱ���ˡ��л�����һ����֧������������ں���ķ�֧�����л��ٴ�����`git checkout`����*

#Զ�ֿ̲�
##���Զ�̿�
�����Լ��git�������������и�������`github`,�����������洴��һ��`repository`���������Ҵ�����`repository`��

![](../image/git-remote.png)

�����ؿ��Զ�̿����
```
$ git remote add origin git@github.com:earthlyfisher/java-book.git
```
��Ӻ�Զ�̿�����־���origin������GitĬ�ϵĽз���Ҳ���Ըĳɱ�ģ�����origin�������һ����֪����Զ�̿⡣

���ϱ��뽫`SSH Key��Կ`��ӵ�`github`�˻��б���.

��һ�������������е��������͵�Զ�ֿ̲�:
```
$ git push -u origin master
```
�ѱ��ؿ���������͵�Զ�̣���`git push`���ʵ�����ǰѵ�ǰ��֧`master`���͵�Զ�̡�

����Զ�̿��ǿյģ����ǵ�һ������`master`��֧ʱ��������`-u`������`Git`������ѱ��ص�`master`��֧�������͵�Զ���µ�`master`��֧������ѱ��ص�`master`��֧��Զ�̵�`master`��֧�������������Ժ�����ͻ�����ȡʱ�Ϳ��Լ����

ֻҪ�������ϲ��������Ժ�Ĳ�����,ֻҪ���������ύ���Ϳ���ͨ�����
```
$ git push origin master
```
##��Զ�̿��¡
����Զ�̿��Ѿ�����������ˣ�����ͨ����¡�ķ�ʽ��Զ�̿��¡�����صĹ�����:
```
$ git clone git@github.com:earthlyfisher/java-book.git
```

**ע�⣺**`Git`֧�ֶ���Э�飬����`https`����ͨ��`ssh`֧�ֵ�ԭ��`git`Э���ٶ���졣

#��֧����
##�޸��ݴ�
���Խ���ǰ���޸�ͨ��`git stash`���������������������޸ĺ��ٻ�������:
�ݴ棺
```
$ git stash
Saved working directory and index state WIP on dev: 97ce1a4 append
HEAD is now at 97ce1a4 append
```
�ݴ��ǰ��֧�Ϳ������仯��.

�鿴�ݴ����б�
```
$ git stash list
stash@{0}: WIP on dev: 97ce1a4 append
stash@{1}: WIP on master: 97ce1a4 append
```
�ָ��ݴ�������:
```
$ git stash pop
On branch dev
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   13.txt
```
`git stash pop`�ָ���ͬʱ��`stash`����Ҳɾ��.
##����Э��
һ������Ŀ����Ҫ`master`,`dev`,`bug`,`feature`��֧��Ϊ�����Ŀ���,����֧�Ĺ������£�
* master��֧������֧�����Ҫʱ����Զ��ͬ����
* dev��֧�ǿ�����֧���Ŷ����г�Ա����Ҫ�����湤��������Ҳ��Ҫ��Զ��ͬ����
* bug��ֻ֧�����ڱ����޸�bug����û��Ҫ�Ƶ�Զ���ˣ������ϰ�Ҫ������ÿ�ܵ����޸��˼���bug��
* feature��֧�Ƿ��Ƶ�Զ�̣�ȡ�������Ƿ�����С�����������濪��