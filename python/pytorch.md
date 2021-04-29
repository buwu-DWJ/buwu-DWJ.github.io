# pytorch  

创建网络的一种快捷方法：Sequential
```python
net = torch.nn.Sequential(
        torch.nn.Linear(STATE_SIZE, HIDDEN_SIZE),
        torch.nn.ReLU(),
        torch.nn.Linear(HIDDEN_SIZE, ACTION_SIZE),
        )
```