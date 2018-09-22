FROM microsoft/dotnet:2.1-sdk

WORKDIR /app

COPY . .

RUN dotnet restore

RUN dotnet build

RUN mkdir /dist

RUN dotnet publish -o /dist

WORKDIR /dist

ENV ASPNETCORE_ENVIRONMENT=Development

ENTRYPOINT [ "dotnet", "Api.dll"]


