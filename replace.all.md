# 替换所有

```bash
git clone https://github.com/bitshares/bitshares-core.git
cd bitshares-core && git checkout 2.0.180612
sed -i  "s/\"BTS/\"RUI/g" `grep -rl BTS .`

sed -i  "s/\"BTS/\"RUI/g" `grep -rl BTS . | grep -v node_modules | grep -v git`
sed -i  "s/\"GPH/\"RUI/g" `grep -rl GPH . | grep -v node_modules | grep -v git`
cd node_modules/bitsharesjs-ws/
sed -i  "s/\"BTS/\"RUI/g" `grep -rl BTS .`
sed -i  "s/\"GPH/\"RUI/g" `grep -rl GPH .`
```
