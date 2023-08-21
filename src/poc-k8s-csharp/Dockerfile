FROM mcr.microsoft.com/dotnet/aspnet:7.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build
WORKDIR /src
COPY ["src/poc-k8s-csharp/poc-k8s-csharp.csproj", "poc-k8s-csharp/"]
RUN dotnet restore "poc-k8s-csharp/poc-k8s-csharp.csproj"
COPY src/ .
WORKDIR "/src/poc-k8s-csharp"
RUN dotnet build "poc-k8s-csharp.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "poc-k8s-csharp.csproj" -c Release -o /app/publish /p:UseAppHost=false

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "poc-k8s-csharp.dll"]