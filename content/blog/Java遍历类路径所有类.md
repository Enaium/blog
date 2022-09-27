---
title: "Java遍历类路径所有类"
date: 2021-07-16T09:58:00+0800
categories: java
---

```java
public class WalkClasspathAllClasses {
    public static void main(String[] args) throws URISyntaxException, IOException {
        List<URL> urls = new ArrayList<>();
        //获取Classpath
        if (WalkClasspathAllClasses.class.getClassLoader() instanceof URLClassLoader) {
            Collections.addAll(urls, ((URLClassLoader) WalkClasspathAllClasses.class.getClassLoader()).getURLs());
        } else {
            for (String s : System.getProperty("java.class.path").split(";")) {
                urls.add(new File(s).toURI().toURL());
            }
        }
        //遍历所有类
        walkAllClasses(urls).forEach(System.out::println);
    }

    private static Set<String> walkAllClasses(List<URL> urls) throws URISyntaxException, IOException {
        Set<String> classes = new HashSet<>();
        for (URL url : urls) {
            if (url.toURI().getScheme().equals("file")) {//判断Scheme是不是file
                File file = new File(url.toURI());
                if (!file.exists()) continue;//是存在
                if (file.isDirectory()) {//如果是目录
                    Files.walkFileTree(file.toPath(), new SimpleFileVisitor<Path>() {//遍历所有文件
                        @Override
                        public FileVisitResult visitFile(Path path, BasicFileAttributes attrs) throws IOException {
                            if (path.toFile().getName().endsWith(".class")) {//判断是否为class后缀
                                //去除路径截取包名
                                String substring = path.toFile().getAbsolutePath().substring(file.getAbsolutePath().length());
                                if (substring.startsWith(File.separator)) {
                                    substring = substring.substring(1);
                                }
                                substring = substring.substring(0, substring.length() - 6);//去掉后缀
                                classes.add(substring.replace(File.separator, "."));//把路径替换为.
                            }
                            return super.visitFile(path, attrs);
                        }
                    });
                } else if (file.getName().endsWith(".jar")) {//如果是jar包
                    JarFile jarFile = new JarFile(file);
                    Enumeration<JarEntry> entries = jarFile.entries();
                    while (entries.hasMoreElements()) {//遍历所有文件
                        JarEntry jarEntry = entries.nextElement();
                        if (!jarEntry.getName().endsWith(".class")) continue;//判断后缀是否为class
                        String replace = jarEntry.getName().replace("/", ".");//把路径替换为.
                        classes.add(replace.substring(0, replace.length() - 6));//去掉后缀
                    }
                }
            }
        }
        return classes;
    }
}
```
