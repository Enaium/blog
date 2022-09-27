---
title: "实现JPanel切换"
data: 2022-2-28 17:13
categories: java
---

```java
public static void main(String[] args) {
    JFrame jFrame = new JFrame("Test");
    jFrame.setSize(500, 500);
    jFrame.setLocationRelativeTo(jFrame.getOwner());
    jFrame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
    jFrame.setLayout(new BorderLayout());
    var jPanel = new JPanel(new BorderLayout());
    
    jFrame.add(jPanel, BorderLayout.CENTER);
    AtomicBoolean b = new AtomicBoolean(false);
    jFrame.add(new JButton("Switch") {
        {
            addActionListener(e -> {
                b.set(!b.get());
                jPanel.removeAll();
                jPanel.repaint();
                jPanel.revalidate();
                jPanel.add(b.get() ? new JPanel() {
                    @Override
                    protected void paintComponent(Graphics g) {
                        setBackground(Color.RED);
                        super.paintComponent(g);
                    }
                } : new JPanel() {
                    @Override
                    protected void paintComponent(Graphics g) {
                        setBackground(Color.GREEN);
                        super.paintComponent(g);
                    }
                });
                jPanel.repaint();
                jPanel.revalidate();
            });
        }
    }, BorderLayout.SOUTH);
    jFrame.setVisible(true);
}
```

主要就是这几行

```java
jPanel.removeAll();//移除全部
jPanel.repaint();//重绘
jPanel.revalidate();//重新验证
jPanel.add();//需要切换的JPanel
jPanel.repaint();
jPanel.revalidate();
```

最新在写JFrame，需要切换多个窗口太麻烦，就直接切换JPanel，最初使用的是CardLayout，限制太多，需要提前把JPanel全部加进去才能切换，后来就用这个方法来动态切换