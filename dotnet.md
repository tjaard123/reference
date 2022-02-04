# dotnet core Nina

## WSL

Ok, this can be painful. There are too many ways to install dotnet. If you stick to apt-get you should be ok.

Gotchas
- If you want to run ASP apps, you need *aspnetcore-runtime*, not just *dotnet-runtime*:

```sh
sudo apt-get install -y aspnetcore-runtime-5.0
```

- If you messed up your install, chances are you now have runtimes in `/usr/local` and `/home/<your-user>`. Even if you don't mess it up this sometimes happens. Now you might run `dotnet run` and see issues on runtime, it's probably because you think it's using the one in usr but it's not. You can solve this by setting `DOTNET_ROOT` env variable. Add it to you .zshrc or whatever you use.

```sh
export DOTNET_ROOT=/usr/share/dotnet
```