创建 venv：
```bash
conda create -n chatglm-6b
conda activate chatglm-6b
```
退出 venv：
```bash
conda deactivate

conda activate base
```
换国内 pip 源：
```bash
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge 
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/msys2/

conda config --set show_channel_urls yes 
```
查看 venv：
```bash
conda env list
```
![image.png](./../assets/1689758880184-b6673119-6a34-49d1-bee1-555ef2a0fd76.png)