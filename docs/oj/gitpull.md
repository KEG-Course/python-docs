# �ֿ���ȡ�̳�

������֮ǰ��ѧϰ�У�����Ѿ������˽������Լ��ֿ��н��� git ���������� `git add`��`git clone`��`git commit`��`git push`��

�����������ڸ��������ʹ�� git ��Ϊ�������˺�������ô��ν��Լ����������˴����Բ���ͻ�ķ�ʽ�ϲ��أ�

---

## ǰ��֪ʶ�ع�

�����Ѿ�������һЩ������ Git ���

* `git clone`����¡Զ�ֿ̲⵽����
* `git add`�����޸���ӵ��ݴ���
* `git commit`���ύ�޸�
* `git push`�����ʹ��뵽Զ�ֿ̲�

������Э��ʱ�����ǻ���ѧ�᣺

* ��Ӷ��Զ�ֿ̲�
* ������Զ�ֿ̲���ȡ����
* �ϲ����طţ�rebase���Ķ�
* �����ͻ

---

## ��¡�����ҵ�ֿ�

```bash
git clone https://git.tsinghua.edu.cn/<git-space>/<repo-name>.git
cd <repo-name>
```

```shell
git clone https://git.tsinghua.edu.cn/python-course-2025/pa2-oj-2025123456.git
Cloning into 'pa2-oj-2025123456'...
remote: Enumerating objects: 3, done.
remote: Counting objects: 100% (3/3), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
Receiving objects: 100% (3/3), done.
```

---

## ���ģ��ֿ�ΪԶ��

���γ��ṩ��ģ��ֿ�����Ϊ `template`��

```bash
git remote add template https://git.tsinghua.edu.cn/python-course-2025/pa2-oj-template.git
```

���������������鿴����Զ�ֿ̲⣺

```bash
git remote -v
```

??? info
    git Ĭ�ϻὫֱ�� clone �����Ĳֿ�Դ������Ϊ `origin`

��Ӧ�ÿ����������

```shell
origin	https://git.tsinghua.edu.cn/python-course-2025/pa2-oj-2025123456.git (fetch)
origin	https://git.tsinghua.edu.cn/python-course-2025/pa2-oj-2025123456.git (push)
template	https://git.tsinghua.edu.cn/python-course-2025/pa2-oj-template.git (fetch)
template	https://git.tsinghua.edu.cn/python-course-2025/pa2-oj-template.git (push)
```

---

## ��ȡģ��ֿ�ĸ���

ִ�У�

```bash
git pull template main
```

������״���ȡ��Git ���ܻ���������Ҫָ���ϲ����ԣ�

```
From https://git.tsinghua.edu.cn/python-course-2025/pa2-oj-template
 * branch            main       -> FETCH_HEAD
hint: You have divergent branches and need to specify how to reconcile them.
hint: You can do so by running one of the following commands sometime before
hint: your next pull:
hint:
hint:   git config pull.rebase false  # merge
hint:   git config pull.rebase true   # rebase
hint:   git config pull.ff only       # fast-forward only
hint:
hint: You can replace "git config" with "git config --global" to set a default
hint: preference for all repositories. You can also pass --rebase, --no-rebase,
hint: or --ff-only on the command line to override the configured default per
hint: invocation.
fatal: Need to specify how to reconcile divergent branches.
```

������ʾѡ��һ�ַ�ʽ���Ƽ�ʹ�� rebase ��ʽ�����ύ��ʷ���ࣩ��

```bash
git config pull.rebase true
```

Ȼ���ٴ���ȡ��

```bash
git pull template main
```

---

## �����ͻ������У�

��ȡģ�����ʱ����������ģ���ж�ͬһ�ļ����� `README.md`���в�ͬ�޸ģ�Git ����ʾ���г�ͻ��

```
From https://git.tsinghua.edu.cn/python-course-2025/pa2-oj-template
 * branch            main       -> FETCH_HEAD
Auto-merging README.md
CONFLICT (add/add): Merge conflict in README.md
error: could not apply fc763b1... Initial commit
hint: Resolve all conflicts manually, mark them as resolved with
hint: "git add/rm <conflicted_files>", then run "git rebase --continue".
hint: You can instead skip this commit: run "git rebase --skip".
hint: To abort and get back to the state before "git rebase", run "git rebase --abort".
hint: Disable this message with "git config advice.mergeConflict false"
Could not apply fc763b1... Initial commit
```

��ʱ��

1. ����ʾ��ͻ���ļ������� `README.md`�����ֶ��޸ĳ�ͻ���ݣ�
2. �޸����ִ�У�

```bash
git add README.md
git rebase --continue
```

�����������ϲ��˴��ύ��Ҳ����������

```bash
git rebase --skip
```

��������ϲ��������ص�ԭ����״̬��

```bash
git rebase --abort
```

---

## ����Ч��

������ϲ������Ĵ���ֿ⽫����ģ����������ݣ������㱾�ش���ϲ����Ժ�ֻ�趨�����У�

```bash
git pull template main
```

���ܱ���ģ�����ݵ�����״̬��

---

## �Զ������ã���ѡ��

��Ҳ��������Ĭ����ȡ��Ϊ������ÿ�ζ���ʾ����

```bash
# ֻ�Ե�ǰ�ֿ�����
git config pull.rebase true

# ���ȫ�����ã����вֿ���Ч��
git config --global pull.rebase true
```

## ��������

- ����㷢���Ѿ���� `Already up to date`�����Լ����Ŀǰ pull ��Դ���Ƿ����Ԥ�ڲֿ⡣����˴� personal �����Լ��ĸ��˲ֿ⡣

![alt text](assets/pull-error.png)